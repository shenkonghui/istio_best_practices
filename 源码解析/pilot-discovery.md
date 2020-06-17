# pilot-discovery

initDNSCerts


- NewServer()
    - initKubeClient()
    - initLeaderElection()
    - initMeshConfiguration()
        - if MeshConfig !=nil
            - NewFixedWatcher()
        - NewWatcher()
        - getMeshConfig()
        - NewFixedWatcher()
    - initMeshNetworks()
    - initCertController()
    - initConfigController()
    - initServiceControllers
    - if EnableCA()
        - s.createCA()
    - initDNSListener()
    - initDiscoveryService()
    - initMonitor()
    - initClusterRegistries()
    - initHTTPSWebhookServer()
    - initSidecarInjector()
    - initConfigValidation()
    - addStartFunc()
        - RunCA()


#### serviceregistry管理       
```
func (s *Server) initServiceControllers(args *PilotArgs) error {
	serviceControllers := s.ServiceController()
	registered := make(map[serviceregistry.ProviderID]bool)
	for _, r := range args.Service.Registries {
		serviceRegistry := serviceregistry.ProviderID(r)
		if _, exists := registered[serviceRegistry]; exists {
			log.Warnf("%s registry specified multiple times.", r)
			continue
		}
		registered[serviceRegistry] = true
		log.Infof("Adding %s registry adapter", serviceRegistry)
		switch serviceRegistry {
		case serviceregistry.Kubernetes:
			if err := s.initKubeRegistry(serviceControllers, args); err != nil {
				return err
			}
		case serviceregistry.Consul:
			if err := s.initConsulRegistry(serviceControllers, args); err != nil {
				return err
			}
		case serviceregistry.Mock:
			s.initMemoryRegistry(serviceControllers)
		default:
			return fmt.Errorf("service registry %s is not supported", r)
		}
	}
```