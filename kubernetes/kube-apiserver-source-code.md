# kube-apiserver

The `kube-apiserver` is the front-end API server for the Kubernetes control plane. It exposes the Kubernetes API, which is used to manage and control Kubernetes clusters. The `kube-apiserver` is responsible for validating and processing API requests, storing API objects in etcd, and responding to API requests from clients.

The `kube-apiserver` implements the Kubernetes API server-side components, including the API resources and controllers. The controllers manage the desired state of the Kubernetes objects, while the API resources expose the Kubernetes objects and their attributes.

The `kube-apiserver` source code is organized into several packages and directories, including `cmd`, `pkg`, and `staging`. The `cmd` directory contains the main program files, including the `apiserver.go` file, which defines the `main()` function. The `pkg` directory contains the core Kubernetes API server-side components, including the API resources and controllers. The `staging` directory contains the experimental API resources and controllers.

Now let's take a closer look at the call stack:

1. The `main()` function is defined in `cmd/kube-apiserver/apiserver.go` and is responsible for starting the `kube-apiserver`.
    
    ```go
    func main() {
    	command := app.NewAPIServerCommand()
    	code := cli.Run(command)
    	os.Exit(code)
    }
    ```
    
2. `main()` calls `app.NewAPIServerCommand()` to create a new command using the `cobra` library.
    
    ```go
    func NewAPIServerCommand() *cobra.Command {
    	s := options.NewServerRunOptions()
    	cmd := &cobra.Command{
    		RunE: func(cmd *cobra.Command, args []string) error {
    			// set default options
    			completedOptions, err := Complete(s)
    			if err != nil {
    				return err
    			}
    			// validate options
    			if errs := completedOptions.Validate(); len(errs) != 0 {
    				return utilerrors.NewAggregate(errs)
    			}
    			// add feature enablement metrics
    			utilfeature.DefaultMutableFeatureGate.AddMetrics()
    			return Run(completedOptions, genericapiserver.SetupSignalHandler())
    		},
    	}
    	return cmd
    }
    ```
    
    1. Complete set the default ServerRunOptions
        
        ```go
        func Complete(s *options.ServerRunOptions) (completedServerRunOptions, error) {
        	var options completedServerRunOptions
        
        	if s.Etcd.EnableWatchCache {
        		sizes := kubeapiserver.DefaultWatchCacheSizes()
        		// Ensure that overrides parse correctly.
        		userSpecified, err := serveroptions.ParseWatchCacheSizes(s.Etcd.WatchCacheSizes)
        		if err != nil {
        			return options, err
        		}
        		for resource, size := range userSpecified {
        			sizes[resource] = size
        		}
        		s.Etcd.WatchCacheSizes, err = serveroptions.WriteWatchCacheSizes(sizes)
        		if err != nil {
        			return options, err
        		}
        	}
        
        	options.ServerRunOptions = s
        	return options, nil
        }
        ```
        
    
3. The `cli.Run()` function is called with the newly created command as an argument, which starts the API server.
4. `Run()` calls `CreateServerChain()` to create a new server chain, which is a chain of middleware functions that are executed for each API request.
    
    ```go
    func Run(completeOptions completedServerRunOptions, stopCh <-chan struct{}) error {
    	// To help debugging, immediately log version
    	klog.Infof("Version: %+v", version.Get())
    
    	klog.InfoS("Golang settings", "GOGC", os.Getenv("GOGC"), "GOMAXPROCS", os.Getenv("GOMAXPROCS"), "GOTRACEBACK", os.Getenv("GOTRACEBACK"))
    
    	server, err := CreateServerChain(completeOptions)
    	if err != nil {
    		return err
    	}
    
    	prepared, err := server.PrepareRun()
    	if err != nil {
    		return err
    	}
    
    	return prepared.Run(stopCh)
    }
    ```
    
5. `CreateServerChain()` calls `CreateKubeAPIServerConfig()` to create a new Kube API server configuration, which contains the configuration for the API server.
    
    ```go
    func CreateServerChain(completedOptions completedServerRunOptions) (*aggregatorapiserver.APIAggregator, error) {
    	kubeAPIServerConfig, serviceResolver, pluginInitializer, err := CreateKubeAPIServerConfig(completedOptions)
    	if err != nil {
    		return nil, err
    	}
    
    	// If additional API servers are added, they should be gated.
    	apiExtensionsConfig, err := createAPIExtensionsConfig(*kubeAPIServerConfig.GenericConfig, kubeAPIServerConfig.ExtraConfig.VersionedInformers, pluginInitializer, completedOptions.ServerRunOptions, completedOptions.MasterCount, serviceResolver, webhook.NewDefaultAuthenticationInfoResolverWrapper(kubeAPIServerConfig.ExtraConfig.ProxyTransport, kubeAPIServerConfig.GenericConfig.EgressSelector, kubeAPIServerConfig.GenericConfig.LoopbackClientConfig, kubeAPIServerConfig.GenericConfig.TracerProvider))
    	if err != nil {
    		return nil, err
    	}
    	crdAPIEnabled := apiExtensionsConfig.GenericConfig.MergedResourceConfig.ResourceEnabled(apiextensionsv1.SchemeGroupVersion.WithResource("customresourcedefinitions"))
    
    	notFoundHandler := notfoundhandler.New(kubeAPIServerConfig.GenericConfig.Serializer, genericapifilters.NoMuxAndDiscoveryIncompleteKey)
    	apiExtensionsServer, err := createAPIExtensionsServer(apiExtensionsConfig, genericapiserver.NewEmptyDelegateWithCustomHandler(notFoundHandler))
    	if err != nil {
    		return nil, err
    	}
    
    	kubeAPIServer, err := CreateKubeAPIServer(kubeAPIServerConfig, apiExtensionsServer.GenericAPIServer)
    	if err != nil {
    		return nil, err
    	}
    
    	// aggregator comes last in the chain
    	aggregatorConfig, err := createAggregatorConfig(*kubeAPIServerConfig.GenericConfig, completedOptions.ServerRunOptions, kubeAPIServerConfig.ExtraConfig.VersionedInformers, serviceResolver, kubeAPIServerConfig.ExtraConfig.ProxyTransport, pluginInitializer)
    	if err != nil {
    		return nil, err
    	}
    	aggregatorServer, err := createAggregatorServer(aggregatorConfig, kubeAPIServer.GenericAPIServer, apiExtensionsServer.Informers, crdAPIEnabled)
    	if err != nil {
    		// we don't need special handling for innerStopCh because the aggregator server doesn't create any go routines
    		return nil, err
    	}
    
    	return aggregatorServer, nil
    }
    ```
    
6. `CreateKubeAPIServerConfig()` calls `buildGenericConfig()` to build a generic configuration for the API server, which includes the API server's codec factory, API resource configuration source, and etcd access path.
    
    ```go
    // CreateKubeAPIServerConfig creates all the resources for running the API server, but runs none of them
    func CreateKubeAPIServerConfig(s completedServerRunOptions) (
    	*controlplane.Config,
    	aggregatorapiserver.ServiceResolver,
    	[]admission.PluginInitializer,
    	error,
    ) {
    	proxyTransport := CreateProxyTransport()
    
    	genericConfig, versionedInformers, serviceResolver, pluginInitializers, admissionPostStartHook, storageFactory, err := buildGenericConfig(s.ServerRunOptions, proxyTransport)
    	if err != nil {
    		return nil, nil, nil, err
    	}
    
    	capabilities.Setup(s.AllowPrivileged, s.MaxConnectionBytesPerSec)
    
    	s.Metrics.Apply()
    	serviceaccount.RegisterMetrics()
    
    	config := &controlplane.Config{
    		GenericConfig: genericConfig,
    		ExtraConfig: controlplane.ExtraConfig{
    			APIResourceConfigSource: storageFactory.APIResourceConfigSource,
    			StorageFactory:          storageFactory,
    			EventTTL:                s.EventTTL,
    			KubeletClientConfig:     s.KubeletConfig,
    			EnableLogsSupport:       s.EnableLogsHandler,
    			ProxyTransport:          proxyTransport,
    
    			ServiceIPRange:          s.PrimaryServiceClusterIPRange,
    			APIServerServiceIP:      s.APIServerServiceIP,
    			SecondaryServiceIPRange: s.SecondaryServiceClusterIPRange,
    
    			APIServerServicePort: 443,
    
    			ServiceNodePortRange:      s.ServiceNodePortRange,
    			KubernetesServiceNodePort: s.KubernetesServiceNodePort,
    
    			EndpointReconcilerType: reconcilers.Type(s.EndpointReconcilerType),
    			MasterCount:            s.MasterCount,
    
    			ServiceAccountIssuer:        s.ServiceAccountIssuer,
    			ServiceAccountMaxExpiration: s.ServiceAccountTokenMaxExpiration,
    			ExtendExpiration:            s.Authentication.ServiceAccounts.ExtendExpiration,
    
    			VersionedInformers: versionedInformers,
    		},
    	}
    
    	clientCAProvider, err := s.Authentication.ClientCert.GetClientCAContentProvider()
    	if err != nil {
    		return nil, nil, nil, err
    	}
    	config.ExtraConfig.ClusterAuthenticationInfo.ClientCA = clientCAProvider
    
    	requestHeaderConfig, err := s.Authentication.RequestHeader.ToAuthenticationRequestHeaderConfig()
    	if err != nil {
    		return nil, nil, nil, err
    	}
    	if requestHeaderConfig != nil {
    		config.ExtraConfig.ClusterAuthenticationInfo.RequestHeaderCA = requestHeaderConfig.CAContentProvider
    		config.ExtraConfig.ClusterAuthenticationInfo.RequestHeaderAllowedNames = requestHeaderConfig.AllowedClientNames
    		config.ExtraConfig.ClusterAuthenticationInfo.RequestHeaderExtraHeaderPrefixes = requestHeaderConfig.ExtraHeaderPrefixes
    		config.ExtraConfig.ClusterAuthenticationInfo.RequestHeaderGroupHeaders = requestHeaderConfig.GroupHeaders
    		config.ExtraConfig.ClusterAuthenticationInfo.RequestHeaderUsernameHeaders = requestHeaderConfig.UsernameHeaders
    	}
    
    	if err := config.GenericConfig.AddPostStartHook("start-kube-apiserver-admission-initializer", admissionPostStartHook); err != nil {
    		return nil, nil, nil, err
    	}
    
    	if config.GenericConfig.EgressSelector != nil {
    		// Use the config.GenericConfig.EgressSelector lookup to find the dialer to connect to the kubelet
    		config.ExtraConfig.KubeletClientConfig.Lookup = config.GenericConfig.EgressSelector.Lookup
    
    		// Use the config.GenericConfig.EgressSelector lookup as the transport used by the "proxy" subresources.
    		networkContext := egressselector.Cluster.AsNetworkContext()
    		dialer, err := config.GenericConfig.EgressSelector.Lookup(networkContext)
    		if err != nil {
    			return nil, nil, nil, err
    		}
    		c := proxyTransport.Clone()
    		c.DialContext = dialer
    		config.ExtraConfig.ProxyTransport = c
    	}
    
    	// Load the public keys.
    	var pubKeys []interface{}
    	for _, f := range s.Authentication.ServiceAccounts.KeyFiles {
    		keys, err := keyutil.PublicKeysFromFile(f)
    		if err != nil {
    			return nil, nil, nil, fmt.Errorf("failed to parse key file %q: %v", f, err)
    		}
    		pubKeys = append(pubKeys, keys...)
    	}
    	// Plumb the required metadata through ExtraConfig.
    	config.ExtraConfig.ServiceAccountIssuerURL = s.Authentication.ServiceAccounts.Issuers[0]
    	config.ExtraConfig.ServiceAccountJWKSURI = s.Authentication.ServiceAccounts.JWKSURI
    	config.ExtraConfig.ServiceAccountPublicKeys = pubKeys
    
    	return config, serviceResolver, pluginInitializers, nil
    }
    ```
    
7. `buildGenericConfig()` creates a codec factory for encoding and decoding, sets up the API resource configuration source, and registers an access path in etcd for all Kubernetes objects.
8. `buildGenericConfig()` then sets up authentication, authorization, and admission plugins, which are used to validate and process API requests.
9. Finally, `buildGenericConfig()` calls `installAPI()` to register the generic API handler, which handles non-resource endpoints such as `/healthz`, `/metrics`, and `/version`.
    
    ```go
    func buildGenericConfig(
    	s *options.ServerRunOptions,
    	proxyTransport *http.Transport,
    ) (
    	genericConfig *genericapiserver.Config,
    	versionedInformers clientgoinformers.SharedInformerFactory,
    	serviceResolver aggregatorapiserver.ServiceResolver,
    	pluginInitializers []admission.PluginInitializer,
    	admissionPostStartHook genericapiserver.PostStartHookFunc,
    	storageFactory *serverstorage.DefaultStorageFactory,
    	lastErr error,
    ) {
    	genericConfig = genericapiserver.NewConfig(legacyscheme.Codecs)
    	genericConfig.MergedResourceConfig = controlplane.DefaultAPIResourceConfigSource()
    	// wrap the definitions to revert any changes from disabled features
    	
    	// etcd
    	storageFactoryConfig := kubeapiserver.NewStorageFactoryConfig()
    	storageFactoryConfig.APIResourceConfig = genericConfig.MergedResourceConfig
    	storageFactory, lastErr = storageFactoryConfig.Complete(s.Etcd).New()
    	if lastErr != nil {
    		return
    	}
    	if lastErr = s.Etcd.ApplyWithStorageFactoryTo(storageFactory, genericConfig); lastErr != nil {
    		return
    	}
    	
    	// Authentication
    	// Authentication.ApplyTo requires already applied OpenAPIConfig and EgressSelector if present
    	if lastErr = s.Authentication.ApplyTo(&genericConfig.Authentication, genericConfig.SecureServing, genericConfig.EgressSelector, genericConfig.OpenAPIConfig, genericConfig.OpenAPIV3Config, clientgoExternalClient, versionedInformers); lastErr != nil {
    		return
    	}
    
    	// Authorization
    	genericConfig.Authorization.Authorizer, genericConfig.RuleResolver, err = BuildAuthorizer(s, genericConfig.EgressSelector, versionedInformers)
    	if err != nil {
    		lastErr = fmt.Errorf("invalid authorization config: %v", err)
    		return
    	}
    	if !sets.NewString(s.Authorization.Modes...).Has(modes.ModeRBAC) {
    		genericConfig.DisabledPostStartHooks.Insert(rbacrest.PostStartHookName)
    	}
    
    	lastErr = s.Audit.ApplyTo(genericConfig)
    	if lastErr != nil {
    		return
    	}
    	
    	// admission
    	admissionConfig := &kubeapiserveradmission.Config{
    		ExternalInformers:    versionedInformers,
    		LoopbackClientConfig: genericConfig.LoopbackClientConfig,
    		CloudConfigFile:      s.CloudProvider.CloudConfigFile,
    	}
    	serviceResolver = buildServiceResolver(s.EnableAggregatorRouting, genericConfig.LoopbackClientConfig.Host, versionedInformers)
    	schemaResolver := resolver.NewDefinitionsSchemaResolver(k8sscheme.Scheme, genericConfig.OpenAPIConfig.GetDefinitions)
    	pluginInitializers, admissionPostStartHook, err = admissionConfig.New(proxyTransport, genericConfig.EgressSelector, serviceResolver, genericConfig.TracerProvider, schemaResolver)
    	if err != nil {
    		lastErr = fmt.Errorf("failed to create admission plugin initializer: %v", err)
    		return
    	}
    
    	dynamicExternalClient, err := dynamic.NewForConfig(kubeClientConfig)
    	if err != nil {
    		lastErr = fmt.Errorf("failed to create real dynamic external client: %w", err)
    		return
    	}
    
    	err = s.Admission.ApplyTo(
    		genericConfig,
    		versionedInformers,
    		clientgoExternalClient,
    		dynamicExternalClient,
    		utilfeature.DefaultFeatureGate,
    		pluginInitializers...)
    	if err != nil {
    		lastErr = fmt.Errorf("failed to initialize admission: %v", err)
    		return
    	}
    
    	return
    }
    ```
    
10. `CreateKubeAPIServerConfig()` then calls `CreateAPIExtensionsConfig()` to create a new API extensions configuration, which contains the configuration for the API extensions server.
11. `CreateAPIExtensionsConfig()` calls `createAPIExtensionsServer()` to create a new API extensions server, which handles the API requests for the API extensions.
12. `createAPIExtensionsServer()` sets up health checks and registers the generic API handler.
13. `CreateKubeAPIServerConfig()` then calls `CreateKubeAPIServer()` to create a new Kube API server, which contains the configuration for the core Kubernetes API server-side components.
14. `CreateKubeAPIServer()` installs the legacy API, installs the core group API, and registers the API handler, which handles the API requests for the core Kubernetes API server-side components.
15. Finally, `CreateKubeAPIServer()` calls `createAggregatorServer()` to create an aggregator server, which handles the API requests for the aggregated API resources.
16. `createAggregatorServer()` registers the API handler and sets up health checks.
