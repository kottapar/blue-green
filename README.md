
It's been about 18 months since we implemented a blue-green deployment model using namespace separation in an AWS EKS cluster. It has been quite some time since I wrote and also I wanted to use github pages instead of Medium. So I wanted to write about how we implemented this. 
There's no one way to implement these things, but this is what we did at that time considering our architecture design and other constraints. 

In order to explain this let's consider a hypothetical mobile app, myapp which is used for making utility payments. It has the usual APIs for user login, user profile and payment etc. It will also make calls to Mastercard and Visa for processing the payments. Each of these will be served by their corresponding microservices; In this case let's assume them to be payment, user-login, user-profile and so on. 
We have both iOS and Android app and so we'd require FQDNs for the APIs to talk to. Let's call our FQDN as myapp.com. 

In AWS we had a Hub and Spoke architecture implemented wherein all the ingress and egress traffic is handled by a VPC. Let's call it as a security account. We had VPCs for prod and dev. The dev hosted the UAT and dev EKS clusters. 

For the initial release of the app we went with a single namespace that hosted the microservices and provided the services. We knew we'd have to upgrade the deployment design but a problem with the deployment of the next version showed why it is important that we do it quickly.

I tried creating the architecture to discuss this. All the API calls from the mobile app will hit the security account where the traffic is segregated. The prod traffic is sent to the prod load-balancer which is backed by two Nginx instances for high availability. Yes, we could've used AWS API gateway service as well. We choose this for the sake of portability :-) It is then sent to the ingress ALB which directs the traffic based on the intended microservice. The microservices talk to a MongoDB database.

![](/before-blue-green.jpg)

### Why we moved to a blue-green deployment
As you might know, with payment apps it is not possible to test all scenarios completely in the UAT and BAT phases. We'll be mostly working with a stub provided by the payment operator. So some scenarios can only be tested when the code is deployed to prod. These will mostly be payment related scenarios. This is fine for the inital release as we'll not be having customers accessing it. But once you go live, we'll have to consider the below risks. If the deployed V2 code is botched:

1. We'll risk the customer accessing this code and thereby customer impact
2. If there's a frontend code change, then we'll lose valuable time correcting and uploading the apps for approval
3. Downtime of the app itself while we determine the cause of the issue
4. Repeat this cycle again

### Proposed changes
So we sat down with the architecture team and our AWS vendors to come up with the below design. Here's what we did. We noticed that the current EKS cluster isn't loaded and so decided to implement this via namespace segregation instead of creating another EKS cluster. 

1. created a new subdomain preprod.myapp.com
2. created two new namespaces blue and green with their corresponding ingress controllers and ALBs
3. The configs in the Nginx instances were seperated for the namespaces
4. As the HA Nginx is the API gateway, we'll control the direction of the prod traffic there
5. 


![](/blue-green.jpg)