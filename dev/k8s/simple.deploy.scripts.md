# k8s image apply scripts

## apply.sh

```bash
#!/bin/bash

# =========================
# 변수 체크
# =========================
if [ -z "$1" ]; then
  echo "build_number를 입력하세요"
  echo "예: ./apply.sh 10"
  exit 1
fi

export BUILD_NUMBER=$1
NAMESPACE="console"
DEPLOYMENT_NAME="my-application"
LABEL_SELECTOR="app=my-application"

echo "BUILD_NUMBER=${BUILD_NUMBER}"
echo "Applying Kubernetes manifests..."

# =========================
# 배포 적용
# =========================
envsubst < deploy.yml | kubectl apply -f -
kubectl apply -f service.yml

# =========================
# Rollout 상태 확인
# =========================
echo "Deployment rollout 확인 중..."
kubectl -n ${NAMESPACE} rollout status deployment/${DEPLOYMENT_NAME}

if [ $? -ne 0 ]; then
  echo "Deployment rollout 실패"
  exit 1
fi

# =========================
# Pod Running 상태 확인
# =========================
echo "Pod status 확인 중..."

POD_STATUS=$(kubectl -n ${NAMESPACE} get pod -l ${LABEL_SELECTOR} \
  -o jsonpath='{.items[0].status.phase}')

if [ "${POD_STATUS}" != "Running" ]; then
  echo "Pod 상태 비정상: ${POD_STATUS}"
  kubectl -n ${NAMESPACE} get pod -l ${LABEL_SELECTOR} -o wide
  exit 1
fi

# =========================
# 최종 정보 출력
# =========================
echo "Pod 정상 Running 상태"
kubectl -n ${NAMESPACE} get pod -l ${LABEL_SELECTOR} -o wide
kubectl -n ${NAMESPACE} get svc ${DEPLOYMENT_NAME}

echo "배포 완료 (image tag: 1.0.${BUILD_NUMBER})"
```

## deploy.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-application
  namespace: console
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-application
  template:
    metadata:
      labels:
        app: my-application
    spec:
      containers:
        - name: my-application
          image: image.repos.com/path/to/image/my-application:1.0.${BUILD_NUMBER}
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_PROFILES_ACTIVE
              value: product
            - name: PLATFORM_GROUP_SVC_NAME
              value: platform-group
```

## service.yml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-application
  namespace: console
spec:
  type: NodePort
  selector:
    app: my-application
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30080
```