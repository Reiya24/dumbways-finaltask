masuk ke dashboard grafana

masukan username admin, dan password admin
![](.14setup_monitoring_images/ba1d670b.png)

ubah password
![](.14setup_monitoring_images/7a8528e0.png)

masuk ke setting > configuration 
klik add datasource
![](.14setup_monitoring_images/d6b8b8fe.png)

pilih prometheus

masukan url, edit form sesuai kebutuhan, klik klik save & test
![](.14setup_monitoring_images/d46b6129.png)

# setup dashboard

klik ikon kotak, pilih new dashboard
![](.14setup_monitoring_images/4a440da9.png)

pilih add a new panel

klik code, masukan rumus query, untuk penggunaan CPU
```shell
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle", instance="IP:9100"}[5m])) * 100 ) 
```

untuk penggunaan memory
```shell
100 * ((node_memory_MemTotal_bytes{instance="10.116.106.150:9100"} - node_memory_MemFree_bytes{instance="10.116.106.150:9100"} - node_memory_Buffers_bytes{instance="10.116.106.150:9100"} - node_memory_Cached_bytes{instance="10.116.106.150:9100"}) / node_memory_MemTotal_bytes{instance="10.116.106.150:9100"})
```

untuk free disk space
```shell
node_filesystem_avail_bytes{instance="10.116.106.170:9100"} / 1073741824
```
![](.14setup_monitoring_images/1d8b01b1.png)

masukan title panel, klik apply

save dashboard
![](.14setup_monitoring_images/1d3b25a4.png)

# alerting
## setup integrasi dengan slack dengan grafana
masuk ke https://api.slack.com , pilih create an app 
![](.14setup_monitoring_images/aaccfce0.png)

from strach
![](.14setup_monitoring_images/3d261ce8.png)

masukan app name, pilih workspace
![](.14setup_monitoring_images/e9f9eab4.png)

pilih incoming webhook
![](.14setup_monitoring_images/ddc981cc.png)

activate incoming webhook
![](.14setup_monitoring_images/c75e4496.png)

add new webhook to workspace
![](.14setup_monitoring_images/123042af.png)

pilih workspace slack
![](.14setup_monitoring_images/f919f375.png)

copy webhook url
![](.14setup_monitoring_images/546a34a4.png)

pada dashboard grafana, klik alerting, notification policies
![](.14setup_monitoring_images/3a369013.png)

pada root policies, pilih edit > create a contact point masukan nama, contact 
point types, webhook url, klik test 
![](.14setup_monitoring_images/68241b29.png)

integrasi berhasil
![](.14setup_monitoring_images/5dd347ad.png)

save contact point
![](.14setup_monitoring_images/869684d4.png)

ubah default for all alert ke slack
![](.14setup_monitoring_images/cbab3875.png)

# membuat alert
edit panel, pilih alert, create alert rule from this panel
![](.14setup_monitoring_images/2a0db2d3.png)

![](.14setup_monitoring_images/31fb6466.png)

save and quit

setup alert berhasil
![](.14setup_monitoring_images/9cd036c6.png)