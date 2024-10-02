# 📌 Spring App Kubernetes Service 배포
![image12](https://github.com/user-attachments/assets/5fac71e4-f859-4374-a710-f284af6b8365)

## 🏓 개요

Kubernetes Pod에 Spring App을 배포합니다. Deploy를 통하여 트래픽을 각 파드에 로드벨런싱될 수 있도록 설정하고, 레플리카 설정을 통해 고가용성을 유지할 수 있도록 합니다.

## ⚙ Spring Boot Rest API
![2024-10-02 12 24 13](https://github.com/user-attachments/assets/08d9c49d-57a0-41c8-98d6-f4ca50deac9b)
사용자가 접속했을 때 로드벨런서가 각 스프링에 접속했음을 확인할 TEST API를 작성합니다.


## ⚙ DockerFile Build and Push
![2024-10-02 11 50 59](https://github.com/user-attachments/assets/ca1f7ab6-ef54-4722-81b6-9ec8267e6fa6)
이미지 파일의 크기를 최적화하기 위해 alpine을 사용하여 Dockerfile을 빌드합니다.


![2024-10-02 12 28 22](https://github.com/user-attachments/assets/466a2e9f-4599-451a-8354-a754a9b1720b)
해당 이미지를 도커허브에 push 합니다.

## ⚙ Deploy / Service yml 작성
다음 kube cli를 yml로 관리하여 실행을 최적화합니다.
```shell
# Deployment 생성
kubectl create deployment spring --image=gymlet/spring_app:1.0 --replicas=3

# LoadBalancer 타입의 Service 생성
kubectl expose deployment spring --type=LoadBalancer --name=spring-service --port=80 --target-port=8080

# 서비스 상태 확인
kubectl get services spring-service

# 모든 서비스 확인
kubectl get services

# 서비스 상세 정보 확인
kubectl describe service spring-service
```


**[spring-service.yaml]**
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring
  labels:
    app: spring
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring
  template:
    metadata:
      labels:
        app: spring
    spec:
      containers:
        - name: spring-container
          image: gymlet/spring_app:1.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m"
            limits:
              memory: "1Gi"
              cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: spring-service
  labels:
    app: spring
spec:
  type: LoadBalancer
  selector:
    app: spring
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

```
## 🎠yml 파일 적용
```shell
kubectl apply -f spring-service.yaml
```

## 🎨 Service / Deploy / ReplicaSet / Pod 확인
![2024-10-02 14 06 10](https://github.com/user-attachments/assets/f9a4219f-55b4-4a34-9a02-355a32b1d6bb)


## 🪁minikube Tunnel 실행
![2024-10-02 14 10 16](https://github.com/user-attachments/assets/db229e88-34b0-4be6-93ab-bdbb37a8d290)

## expose 포트포워딩
![2024-10-02 14 16 57](https://github.com/user-attachments/assets/6183786b-e34a-4d3f-b1d1-eb4384d3e54d)
![2024-10-02 14 18 24](https://github.com/user-attachments/assets/470487fb-6fbc-4470-b301-ee6a5a9ca1a1)

**External IP를 사용해서 Deploy에 접근가능 호스트 80번 포트로 접속 시 각 스프링 서버에 로드벨런싱 됩니다.**

## 🎑 결과 확인
**127.0.0.1:80 접속 시 각 스프링 서버에 트래픽이 로드벨런싱된 것을 확인할 수 있습니다.**
![2024-10-02 14 03 46](https://github.com/user-attachments/assets/db0acc35-f26b-46a1-9d55-10c7bc1a8439)

![2024-10-02 14 03 57](https://github.com/user-attachments/assets/960f043c-50dd-4ac0-b94f-d1a5426fd0d0)



## ❗ Trouble Shooting
![2024-10-02 11 00 12](https://github.com/user-attachments/assets/ae80ded2-8c7c-4cf6-8a66-438c712e0861)
노드의 CPU 자원이 부족해서 pod를 3개 올릴 수 없다는 에러가 발생하였습니다.

![2024-10-02 11 00 02](https://github.com/user-attachments/assets/779d6696-7c83-4dfc-816e-b2ac5fc9f021)
htop 명령어를 통해 VM의 CPU 코어수를 확인해보니 2개만 존재했습니다.

![2024-10-02 15 08 52](https://github.com/user-attachments/assets/24fd84bd-28f2-4b01-9bb3-c85695c52b4e)

pod 1개당 cpu를 0.5 코어를 차지하고 있었고, 리소스 사용량을 줄이도록 합니다.

![2024-10-02 15 09 34](https://github.com/user-attachments/assets/606ab190-fcb9-462b-8b22-52bdcc8c3bc6)

pod 1개당 소모하는 리소스를 0.25core, Mem 256MB로 수정합니다.

![2024-10-02 16 02 34](https://github.com/user-attachments/assets/930f58bd-2389-4abc-9333-ba7f84a47d60)
![2024-10-02 16 05 24](https://github.com/user-attachments/assets/366f6638-b462-453f-91d3-abfff8731186)

리소스 수정 후 정상적으로 pod가 Running 되었습니다.

## 📝 결론 및 고찰
쿠버네티스에 서비스를 배포해보면서 Pod, Replicaset, Deploy, Service에 대한 개념과 컨셉을 이해하였습니다. 로드벨런싱을 통해 트래픽을 분산시켜 시스템 안정성을 높이고, 컨트롤 플레인이 파드의 개수 유지를 보장하면서 고가용성을 보장해주는 쿠버네티스의 기능을 학습할 수 있었습니다.