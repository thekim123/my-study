
ECS 에 CI/CD 배포를 해보았다. CI/CD 자체는 간단했다.... 이미 많은 레퍼런스가 있어서 참고하면서 진행했다. 그러나 아침에 혹시 하는 마음에 비용을 조회해보니 5달러가 이미 청구가 되어 있었다. 이유는 내가 Fargate 배포를 했는데 이 Fargate는 프리티어 사용량 같은 개념이 없기 때문이다. 그래서 클러스터의 용량 공급자를 EC2로 변경했다. 그리고 이 부분에서 어려움이 많이 생겼다. 

먼저 ECS의 task들을 IaC로 만들어야 되므로 환경변수들을 task에 저장해두었던 방법을 변경해야 했다. 여기서 나는 AWS의 `SystemManager`에 있는 파라미터 스토어 라는 서비스를 사용했다. 이곳에 파라미터들을 저장하고 AWS내에서 arn을 호출하는 것으로 불러올 수 있었다.

아래는 완성된 task의 json이다.
```json
{  
  "family": "onthelook-task-ecs",  
  "containerDefinitions": [  
    {  
      "name": "onthelook",  
      "image": "789909227789.dkr.ecr.ap-northeast-2.amazonaws.com/on-the-look:latest",  
      "cpu": 0,  
      "portMappings": [  
        {  
          "name": "onthelook-spring-8080",  
          "containerPort": 8080,  
          "hostPort": 8080,  
          "protocol": "tcp",  
          "appProtocol": "http"  
        }  
      ],  
      "essential": true,  
      "secrets": [  
        {  
          "name": "GOOGLE_CLIENT_SECRET",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/GOOGLE_CLIENT_SECRET"  
        },  
        {  
          "name": "KAKAO_CLIENT_SECRET",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/KAKAO_CLIENT_SECRET"  
        },  
        {  
          "name": "KAKAO_REDIRECT_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/KAKAO_REDIRECT_URI"  
        },  
        {  
          "name": "PORT",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/PORT"  
        },  
        {  
          "name": "GOOGLE_CLIENT_ID",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/GOOGLE_CLIENT_ID"  
        },  
        {  
          "name": "GOOGLE_AUTHORIZATION_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/GOOGLE_AUTHORIZATION_URI"  
        },  
        {  
          "name": "KAKAO_JWK_SET_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/KAKAO_JWK_SET_URI"  
        },  
        {  
          "name": "JWT_SECRET_KEY",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/JWT_SECRET_KEY"  
        },  
        {  
          "name": "GOOGLE_TOKEN_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/GOOGLE_TOKEN_URI"  
        },  
        {  
          "name": "KAKAO_TOKEN_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/KAKAO_TOKEN_URI"  
        },  
        {  
          "name": "KAKAO_AUTHORIZATION_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/KAKAO_AUTHORIZATION_URI"  
        },  
        {  
          "name": "DB_USERNAME",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/DB_USERNAME"  
        },  
        {  
          "name": "REFRESH_EXPIRATION_TIME",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/REFRESH_EXPIRATION_TIME"  
        },  
        {  
          "name": "DB_URL",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/DB_URL"  
        },  
        {  
          "name": "KAKAO_CLIENT_ID",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/KAKAO_CLIENT_ID"  
        },  
        {  
          "name": "KAKAO_USER_INFO_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/KAKAO_USER_INFO_URI"  
        },  
        {  
          "name": "GOOGLE_JWK_SET_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/GOOGLE_JWK_SET_URI"  
        },  
        {  
          "name": "ACCESS_EXPIRATION_TIME",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/ACCESS_EXPIRATION_TIME"  
        },  
        {  
          "name": "GOOGLE_USER_INFO_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/GOOGLE_USER_INFO_URI"  
        },  
        {  
          "name": "DB_PASSWORD",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/DB_PASSWORD"  
        },  
        {  
          "name": "GOOGLE_REDIRECT_URI",  
          "valueFrom": "arn:aws:ssm:ap-northeast-2:789909227789:parameter/GOOGLE_REDIRECT_URI"  
        }  
      ],  
      "environmentFiles": [],  
      "mountPoints": [],  
      "volumesFrom": [],  
      "ulimits": [],  
      "logConfiguration": {  
        "logDriver": "awslogs",  
        "options": {  
          "awslogs-group": "/ecs/onthelook-task-ecs",  
          "mode": "non-blocking",  
          "awslogs-create-group": "true",  
          "max-buffer-size": "25m",  
          "awslogs-region": "ap-northeast-2",  
          "awslogs-stream-prefix": "ecs"  
        },  
        "secretOptions": []  
      },  
      "systemControls": []  
    }  
  ],  
  "executionRoleArn": "arn:aws:iam::789909227789:role/ecsTaskExecutionRole",  
  "networkMode": "bridge",  
  "volumes": [],  
  "placementConstraints": [],  
  "requiresCompatibilities": [  
    "EC2"  
  ],  
  "cpu": "512",  
  "memory": "800",  
  "runtimePlatform": {  
    "cpuArchitecture": "X86_64",  
    "operatingSystemFamily": "LINUX"  
  }  
}
```

완성하고 나서 좀 걸리는 부분은 컨테이너 포트와 호스트 포트는 하드코딩되어 있는 것인데 이건 어떻게 해야할지 좀 더 생각해봐야 할 듯 하다. 직접 해보면서 애먹었던 부분은 `cpu`와 `memory`를 설정하는 부분인데... 이게 내가 t2.micro로 하다보니 가용가능한 자원이 열악했다... 그래서 1cpu, 1024 memory같은 용량을 사용하면 배포 시작전부터 ecs에서 이것을 충족하는 숫자로 바꿔오라고 하면서 실패 처리를 한다. 이 부분을 발견하기 까지 시간이 오래 걸렸다. 이유는 ecs가 배포작업 자체를 쌓아놓고 하는데 내가 ci/cd 테스트를 한답시고 여러개의 배포를 쌓아놨기 때문이다. 나중에 답답해서 강제 업데이트를 하고 보니 저런 문제가 있었다는 것을 알게 되었다. 또한 내가 배포 로그를 볼 수 있다는걸 몰랐다는 것도 한 몫 했다.

또 하나의 이슈는 배포 속도가 너무 느리다는 것이다. 지금 ci/cd를 요청하고 38분이 걸려 배포가 완료된다. 이거 어떻게 개선할 수 있을까? 그걸 좀 생각해봐야 할 거 같다. 그전에 대체 왜 이렇게 오래 걸리는걸까? 

이건 알고보니 내가 아래 설정을 저렇게 해놨기 때문이었다. 음 그러니까 1개의 태스크가 실행중이어야 된다는 것이다. 그런데 2개를 실행할 자원은 없으니 기존에 있던것만 계속 실행하고 있는것이다. 배포가 밀리는 것이다. 아마도 이전에 35분이 걸렷던 것은 진행된 이유가 내가 뭔가 강제 업데이트를 막 눌렀기 때문이 아닌가 싶다.
```yaml
minimumHealthyPercent: 100 -> 0
maximumPercent: 200 -> 150(100 초과해야만 함)
```
아무튼... 이 값을 위 설정처럼 바꾸니 배포가 바로바로 시작되었다.

어찌됬든 이번에 뭔가 저 task를 json으로 만들어서 해보는데 좀 의미가 컷다라고 생각한다. IaC에 다가가는 중인건가? 허허


아래는 deploy.yml이다.
```yaml
name: Deploy to ECS  
  
on:  
  push:  
    branches: [main]  # main 브랜치에 push 시 작동  
  
jobs:  
  deploy:  
    runs-on: ubuntu-latest  
  
    steps:  
      - name: Checkout  
        uses: actions/checkout@v3  
  
      - name: Configure AWS Credentials  
        uses: aws-actions/configure-aws-credentials@v2  
        with:  
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}  
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}  
          aws-region: ap-northeast-2  
  
      - name: Log in to Amazon ECR  
        id: login-ecr  
        uses: aws-actions/amazon-ecr-login@v1  
  
      - name: Build, tag, and push image to ECR  
        run: |  
          IMAGE_TAG=latest          ECR_REPO_URI=789909227789.dkr.ecr.ap-northeast-2.amazonaws.com/on-the-look          docker build -t $ECR_REPO_URI:$IMAGE_TAG .          docker push $ECR_REPO_URI:$IMAGE_TAG  
      - name: Deploy to ECS  
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1  
        with:  
          task-definition: ecs-task-def.json  
          service: onthelook-service-dev4  
          cluster: onthelook-ecs  
          wait-for-service-stability: true
```


