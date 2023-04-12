### Node Issue
#### Tools
* [node-problem-detector](https://github.com/kubernetes/node-problem-detector)

#### 问题排查
* ssh到内网节点
	* 创建一个支持ssh的pod
	* 并通过负载均衡器转发ssh请求
* 查看日志
	* systemd log
		* journalctl -afu kubelet -S "2023-04-12 15:39:00"
			-u unit，对应的systemd的组件，如kubelet
			
			-f follow，跟踪最新日志
			
			-a show all，现实所有日志列
			
			-S since，从某一时间开始 -S "***"
			对于标准的容器日志
	* container log
		* kubectl logs -f -c <containername> <podname>
		* kubectl logs -f--all-containers <podname> 
		* kubectl logs -f-c <podname> --previous
		* kubectl exec -it <podname> -- tail -f /path/to/log
	
