   确认这个容器所在的服务器
27  kubectl get pods mongo-856c6c544f-j6t9r -o yaml -n crm | grep -i docker  
containerID: docker://18ccb90e872e1ffbbec04571f6b44c87fa9d924b4839320e9c03a30054e69abd
   34  sudo docker inspect 18ccb90e8 | grep log
