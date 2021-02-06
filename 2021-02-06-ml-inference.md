# Machine Learning Inference
How do companies deploy their ML models to production?

### 당근마켓
- Data Scientist uploads model to S3 -> SRE deploys with TF Serving on ECS
- Researching about KF-Serving (kubeflow)
source: [Kubeflow 파이프라인 운용하기](https://medium.com/daangn/kubeflow-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8-%EC%9A%B4%EC%9A%A9%ED%95%98%EA%B8%B0-6c6d7bc98c30)

### Riiid
- Used to deploy with AWS sagemaker
- Now moved to MLFlow

### PUBG
- ?

### Kakao 뉴스 추천팀?
- HBase + rocks DB cache
