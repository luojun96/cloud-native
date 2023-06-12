# Controller Manager

The Kubernetes controller manager is a daemon that embeds the core control loops shipped with Kubernetes. In applications of robotics and automation, a control loop is a non-terminating loop that regulates the state of the system. In Kubernetes, a controller is a control loop that watches the shared state of the cluster through the apiserver and makes changes attempting to move the current state towards the desired state.

## Source Code

Here is a high-level roadmap for navigating and understanding the source code for `kube-controller-manager`:

- **Starting Point**: The starting point for the kube-controller-manager is the main() function located at `cmd/kube-controller-manager/controller-manager.go`. This function sets up the command line interface and loads the options and configurations needed to start the controller manager.

- **Controllers Initialization**: The `pkg/controller` directory contains the actual controller code. Each controller has its own subdirectory. For example, the code for the Replication Controller is in `pkg/controller/replication`, the Service Controller is in `pkg/controller/service`, and so forth.

- **Controller Interface**: Every controller implements the controller interface, which defines the `Run()` function. This function starts the control loop for that controller. Look at how each controller implements this function for a better understanding of what the controller does.

- **Shared Informers**: Most controllers make use of "Shared Informers" to efficiently watch for changes to resources over the API. Shared Informers are located in `pkg/controller/informers`.

- **Work Queue**s: Work queues are used by the controllers to handle the changes noticed by the informers. This is in the `pkg/util/workqueue directory`.

## The pattern of a controller

- Implements a control loop that repeatedly processes work in a queue.
- Uses shared informers to watch for changes to resources.
- Add the resource key to the queue when a change is detected.
- The control loop processes work in the queue, often in the form of a syncHandler function.

In the typical Kubernetes controller, you would find a Run() method which starts the control loop and contains the logic for handling work in the queue, and other methods for handling events and errors.

Here is a simplified example of what this might look like:

```go
type Controller struct {
    queue workqueue.RateLimitingInterface
    informer cache.SharedIndexInformer
    syncHandler func(string) error
}

func (c *Controller) Run(stopCh <-chan struct{}) {
    defer c.queue.ShutDown()
    
    go c.informer.Run(stopCh)

    for {
        key, quit := c.queue.Get()
        if quit {
            return
        }

        err := c.syncHandler(key.(string))

        c.handleErr(err, key)
    }
}

func (c *Controller) handleErr(err error, key interface{}) {
    if err == nil {
        c.queue.Forget(key)
        return
    }

    if c.queue.NumRequeues(key) < 5 {
        c.queue.AddRateLimited(key)
        return
    }

    c.queue.Forget(key)
    utilruntime.HandleError(err)
}
```
