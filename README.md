
It's been about 18 months since we implemented a blue-green deployment model using namespace separation in an AWS EKS cluster. It has been quite some time since I wrote and also I wanted to use github pages instead of Medium. So I wanted to write about how we implemented this. 
There's no one way to implement these things, but this is what we did at that time considering our architecture design and other constraints. 

In order to explain this let's consider a hypothetical mobile app, myapp which is used for making utility payments. It has the usual APIs for user login, user profile and payment etc. It will also make calls to Mastercard and Visa for processing the payments. Each of these will be served by their corresponding microservices; In this case let's assume them to be payment, user-login, user-profile and so on. 
We have both iOS and Android app and so we'd require FQDNs for the APIs to talk to. Let's call our FQDN as myapp.com. 

In AWS we had a Hub and Spoke architecture implemented wherein all the ingress and egress traffic is handled by a VPC. Let's call it as a security account. We had VPCs for prod and dev. The dev hosted the UAT and dev EKS clusters. 

For the initial release of the app we went with a single namespace that hosted the microservices and provided the services. We knew we'd have to upgrade the deployment design but a problem with the deployment of the next version showed why it is really important. 

![](/before-blue-green.jpg)

### Why we moved to a blue-green deployment
As you might know with payment apps, it is not possible to test all scenarios completely in the UAT. We'll be mostly working with a stub provided by the payment operator. So some scenarios can only be tested when the code is deployed to prod. 

### 

![](/blue-green.jpg)