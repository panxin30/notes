1、稳定且唯一的网络标识符
2、稳定且持久的存储
3、有序、平滑的部署和扩展;
4、有序、平滑地终止和删除;
5、有序的滚动更新;   主从，所有从更新了，再更新主
    三个组件：headless service、StatefulSet、volumeClaimTemplate
podname.servicename.nsname.svc.cluster.local