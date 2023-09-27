- **D2** is a diagram scripting language that turns text to diagrams.
- 리포지토리: https://github.com/terrastruct/d2
-
- 한국인 개발자가 개발한 Dirgrams(https://diagrams.mingrammer.com)는 나름대로 활발하게 사용이 되나 한동안 유지보수가 안되고 있다. **D2**는 더 많은 사용자를 확보했고 golang기반이나 클라우드 아키텍처에 쓰이는 데는 아직까지 불편함이 있다. 그래서 D2를 클라우드 아키텍처 Tool로 적합한지 알아보려고 한다.
-
-
  ```python
  from diagrams import Diagram
  from diagrams.aws.compute import EC2
  from diagrams.aws.database import RDS
  from diagrams.aws.network import ELB
  from diagrams.aws.storage import S3
  
  with Diagram("", show=False):
      ELB("lb") >> EC2("web") >> RDS("userdb") >> S3("store")
  ```
- ![web services diagram](https://diagrams.mingrammer.com/img/web_services_diagram.png)
-
- 이 코드를 아래와 같이 변환해보았다.
-
  ```d2
  direction: right
  
  lb: "lb" {
    shape: image
    icon: https://icon.icepanel.io/AWS/svg/Networking-Content-Delivery/Elastic-Load-Balancing.svg
    width: 100
    height: 100
  }
  
  ec2: "web" {
    shape: image
    icon: https://icon.icepanel.io/AWS/svg/Compute/EC2.svg
    width: 100
    height: 100
  }
  
  db: "db" {
    shape: image
    icon: https://icon.icepanel.io/AWS/svg/Database/RDS.svg
    width: 100
    height: 100
  }
  
  s3: "store" {
    shape: image
    icon: https://icon.icepanel.io/AWS/svg/Storage/Simple-Storage-Service.svg
    width: 100
    height: 100
  }
  
  lb -> ec2 -> db -> s3
  
  ```
- ![out.png](../assets/out_1694498760997_0.png)
-
- 유사하게 만들 수 있으나 다소 사용하기 불편함이 있다.
-
-
  ```python
  from diagrams import Diagram
  from diagrams.aws.compute import EC2
  from diagrams.aws.database import RDS
  from diagrams.aws.network import ELB
  
  with Diagram("Grouped Workers", show=False, direction="TB"):
      ELB("lb") >> [EC2("worker1"),
                    EC2("worker2"),
                    EC2("worker3"),
                    EC2("worker4"),
                    EC2("worker5")] >> RDS("events")
  ```
- https://diagrams.mingrammer.com/img/grouped_workers_diagram.png
-
- ![exmaple2_darge.png](../assets/exmaple2_darge_1694768131276_0.png)
- ![exmaple2_elk.png](../assets/exmaple2_elk_1694768082454_0.png)
-
  ```text
  direction: down
  
  lb: "lb" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FNetworking%20&%20Content%20Delivery%2FElastic-Load-Balancing.svg
    width: 100
    height: 100
  }
  
  worker1: {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-EC2_light-bg.svg
    width: 100
    height: 100
  }
  worker2: {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-EC2_light-bg.svg
    width: 100
    height: 100
  }
  worker3: {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-EC2_light-bg.svg
    width: 100
    height: 100
  }
  worker4: {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-EC2_light-bg.svg
    width: 100
    height: 100
  }
  worker5: {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-EC2_light-bg.svg
    width: 100
    height: 100
  }
  
  db: "events" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FDatabase%2FAmazon-RDS.svg
    width: 100
    height: 100
  }
  
  lb -> worker1 -> db
  lb -> worker2 -> db
  lb -> worker3 -> db
  lb -> worker4 -> db
  lb -> worker5 -> db
  
  ```
-
  ```bash
  d2 -s -t 302 -l elk example2.d2 exmaple2_elk.png
  d2 -s -t 302 -l dagre example2.d2 exmaple2_darge.png
  ```
-
  ```python
  direction: right
  container1: "" {
    web1
    web2
    web3
  }
  
  container2: "" {
    userdb
    userdb ro
  }
  
  dns -> lb
  lb -> container1.web1
  lb -> container1.web2
  lb -> container1.web3
  
  container1.web1 -> container2.userdb
  container1.web2 -> container2.userdb
  container1.web3 -> container2.userdb
  
  container1.web1 -> memcached
  container1.web2 -> memcached
  container1.web3 -> memcached
  
  container2.userdb -- container2.userdb ro
  ```
-
-
-
  ```python
  dns: "dns" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FNetworking%20&%20Content%20Delivery%2FAmazon-Route-53.svg
    width: 100
    height: 100
  }
    
  lb: "lb" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FNetworking%20&%20Content%20Delivery%2FElastic-Load-Balancing.svg
    width: 100
    height: 100
  }
  
  memcached : "memcached" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FDatabase%2FAmazon-ElastiCache.svg
    width: 100
    height: 100
  }
      
  container1.web1: "web1" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-Elastic-Container-Service.svg
    width: 100
    height: 100
  }
   
  container1.web2: "web2" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-Elastic-Container-Service.svg
    width: 100
    height: 100
  }
    
  container1.web3: "web3" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FCompute%2FAmazon-Elastic-Container-Service.svg
    width: 100
    height: 100
  }
  container2.userdb: "uesrdb" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FDatabase%2FAmazon-RDS.svg
    width: 100
    height: 100
  }
  
  container2.userdb ro: "uesrdb ro" {
    shape: image
    icon: https://icons.terrastruct.com/aws%2FDatabase%2FAmazon-RDS.svg
    width: 100
    height: 100
  }
  
  direction: right
  container1: "Services" {
    web1
    web2
    web3
  }
  
  container2: "DB Cluster" {
    userdb
    userdb ro
  }
  
  dns -> lb
  lb -> container1
  container1 -> container2.userdb
  container2.userdb -- container2.userdb ro
  container1 -> memcached
  ```
-
-
