
It's been about 18 months since we implemented a blue-green deployment model using namespace separation in an AWS EKS cluster. It has been quite some time since I wrote and also I wanted to use github pages instead of Medium. So I wanted to write about how we implemented this. 
There's no one way to implement these things, but this is what we did at that time considering our architecture design and other constraints. 

In order to explain this I'll consider a hypothetical mobile app which is used for making utility payments. It has the usual APIs for user login, user profile and payment etc. It will also make calls to Mastercard and Visa for processing the payments. Each of these will be served by their corresponding microservices; In this case let's assume them to be payment, user-login, user-profile and so on. 

![](/blue-green-architecture.jpg)