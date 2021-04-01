### BRPC-RDMA 实现-RDMA接口封装

#### RDMA API能做什么？

- RDMA API允许用户程序直接访问硬件。Mellanox提供动态链接库，通过verbs API来访问硬件。通过verbs允许定制和优化 RDMA的网络。
- 为了执行RDMA操作，通过QP(Queue Pair)机制来完成连接和操作远程的主机。这个和标准的IP协议栈类似。一个QP等同于网络编程中的socket.QP可以在本地和对等连接的主机进行初始化。CM(Communication Manager)可以被用来根据QP优先针对实际QP设置来交换信息
- 一旦QP连接，verbs API可以执行RDMA的read/write/atomic。同时也可以进行序列化的read/write(这个read/write和socket中read/write类似)

####  BRPC如何封装RDMA的库函数?

- BRPC-RDMA是通过加载libibverbs.so和librdmacm.so两个库，获取两个库函数的函数。然后针对从两个库函数重新函数符号，以便在后续进行接口封装，实现必要的读、写、原子操作

  ```c++
  static int ReadRdmaDynamicLib() {
      // Must load libibverbs.so before librdmacm.so
      g_handle_ibverbs = dlopen("libibverbs.so", RTLD_LAZY);
      if (!g_handle_ibverbs) {
          LOG(ERROR) << "Fail to load libibverbs.so due to " << dlerror();
          return -1;
      }
  
      g_handle_rdmacm = dlopen("librdmacm.so", RTLD_LAZY);
      if (!g_handle_rdmacm) {
          LOG(ERROR) << "Fail to load librdmacm.so due to " << dlerror();
          return -1;
      }
  
      LoadSymbol(g_handle_rdmacm, RdmaGetDevices, "rdma_get_devices");
      LoadSymbol(g_handle_rdmacm, RdmaFreeDevices, "rdma_free_devices");
      LoadSymbol(g_handle_rdmacm, RdmaCreateId, "rdma_create_id");
      LoadSymbol(g_handle_rdmacm, RdmaDestroyId, "rdma_destroy_id");
      LoadSymbol(g_handle_rdmacm, RdmaResolveAddr, "rdma_resolve_addr");
      LoadSymbol(g_handle_rdmacm, RdmaBindAddr, "rdma_bind_addr");
      LoadSymbol(g_handle_rdmacm, RdmaResolveRoute, "rdma_resolve_route");
      LoadSymbol(g_handle_rdmacm, RdmaListen, "rdma_listen");
      LoadSymbol(g_handle_rdmacm, RdmaConnect, "rdma_connect");
      LoadSymbol(g_handle_rdmacm, RdmaGetRequest, "rdma_get_request");
      LoadSymbol(g_handle_rdmacm, RdmaAccept, "rdma_accept");
      LoadSymbol(g_handle_rdmacm, RdmaDisconnect, "rdma_disconnect");
      LoadSymbol(g_handle_rdmacm, RdmaGetCmEvent, "rdma_get_cm_event");
      LoadSymbol(g_handle_rdmacm, RdmaAckCmEvent, "rdma_ack_cm_event");
      LoadSymbol(g_handle_rdmacm, RdmaCreateQp, "rdma_create_qp");
  
      LoadSymbol(g_handle_ibverbs, IbvForkInit, "ibv_fork_init");
      LoadSymbol(g_handle_ibverbs, IbvQueryDevice, "ibv_query_device");
      LoadSymbol(g_handle_ibverbs, IbvQueryPort, "ibv_query_port");
      LoadSymbol(g_handle_ibverbs, IbvAllocPd, "ibv_alloc_pd");
      LoadSymbol(g_handle_ibverbs, IbvDeallocPd, "ibv_dealloc_pd");
      LoadSymbol(g_handle_ibverbs, IbvCreateCq, "ibv_create_cq");
      LoadSymbol(g_handle_ibverbs, IbvDestroyCq, "ibv_destroy_cq");
      LoadSymbol(g_handle_ibverbs, IbvCreateCompChannel, "ibv_create_comp_channel");
      LoadSymbol(g_handle_ibverbs, IbvDestroyCompChannel, "ibv_destroy_comp_channel");
      LoadSymbol(g_handle_ibverbs, IbvRegMr, "ibv_reg_mr");
      LoadSymbol(g_handle_ibverbs, IbvDeregMr, "ibv_dereg_mr");
      LoadSymbol(g_handle_ibverbs, IbvGetCqEvent, "ibv_get_cq_event");
      LoadSymbol(g_handle_ibverbs, IbvAckCqEvents, "ibv_ack_cq_events");
      LoadSymbol(g_handle_ibverbs, IbvGetAsyncEvent, "ibv_get_async_event");
      LoadSymbol(g_handle_ibverbs, IbvAckAsyncEvent, "ibv_ack_async_event");
      LoadSymbol(g_handle_ibverbs, IbvDestroyQp, "ibv_destroy_qp");
  
      return 0;
  }
  ```

  
