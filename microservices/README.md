# Explanations

## Services
- activemq    
- eventstore  
- jobseeker   
- recruiter   
- job
- message
- operator
- api
- web

## Resources 
1. First is we provision our AWS resources using terraform. 
    - VPC
    - EKS
    - RDS
    - Route53
    - ACM (SSL)
    - ECR
    - S3 (as terraform backend)
    - IAM (Create user, grant access to ECR and EKS cluster)

2. We use EKS which supports scaling and management using kubernetes container orchestration. We will also deploying ALB to take care of load balancing and ASG for auto scaling nodes.

3. Then we need to create a Dockerfile for each of our services (or one image for BE and FE services).

4. Then we will need to setup CI with steps install, test, build and deploy. Build step will automatically build and push the docker images to our ECR. We will push latest/stable and a numbered version of the same image. The deploy step would connect to our EKS cluster using kubectl.

5. Kubernetes deployments shall use the latest (development) and stable (production) tags at all times. So whenever there is a new build we just need to restart the service affected. We can then restart our kubernetes services by accessing the EKS cluster via CI. Making sure that the AWS credentials are stored as secret environment variables in CI.

6. Once all services is up and running, we define all hosts on an ingress definition file. Which would look like

```
rules:
    - host: domain.com
      http:
        paths:
          - backend:
            serviceName: web
            servicePort: 80

    - host: api.domain.com
      http:
        paths:
          - backend:
            serviceName: api
            servicePort: 80
```

7. After applying the ingress file we will get an ELB address. We will then use it for our Route53 configuration. 

```
❯ kubectl get ingresses

NAME  CLASS    HOSTS                         ADDRESS                                               PORTS     AGE                                                         
main  none   web.domain.com,api.domain.com   default-1854330541.ap-northeast-1.elb.amazonaws.com.  80, 443   308d

```

Route53 entry using terraform
```
resource "aws_route53_record" "api_production" {
  zone_id = data.aws_route53_zone.route53_zone.zone_id
  name    = "api"
  type    = "CNAME"
  ttl     = "300"
  records = ["default-1854330541.ap-northeast-1.elb.amazonaws.com."]
}

```

8. The RDS will be accessible outside our VPC but we will limit access using security group. This is useful for when devs need to check database on staging or development environment.


9. Questions: 

    - Which countries will be the main users for the application? So we can choose the closest aws region.