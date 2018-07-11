# Exercise 0 - Creating an IBM Cloud Account and creating a Kubernetes cluster

In this exercise, you'll create an account, attach a feature (promo) code, and create a Kubernetes cluster. A credit card is not required. This workshop will also work with Paid clusters, with a few differences pointed out along the way. Paid clusters come with support for Kubernetes Ingress via an external loadbalancer.

## Create an IBM Cloud Account

1. Visit this [registration link](https://ibm.biz/BdYB5d) and sign up. Follow futher steps with email validation and so on.

2. Get a feature code from this [spreadsheet](https://docs.google.com/spreadsheets/d/1WRJ0CuFRF8bHdUvo0S1EDrGgOrECSpL9gjVUdbOvxQw/edit?usp=sharing). Be sure to mark which one you took so your neighbor doesn't get an error!

3. Click the "User Icon" in the upper right, then in the dropdown select "Profile & Account"

4. On the left select "Billing"

5. Scroll to the middle of the page where there is an 'Apply Code' button for feature codes. Apply the code. 


## Create a Kubernetes cluster

1. Click the the upper left hamburger menu. Select "Containers". It should be near the top.

2. Select "Create Cluster"

3. If you see a region selection menu, select "US South"

4. Select cluster type "Free"

5. Name your cluster, and press "Create"

You now have kicked off cluster creation. This will take a few minutes, but you can proceed with exercise 1 now, which will help you install the ``ibmcloud`` cli and connect to your kubernetes cluster.

### [Continue to Exercise 1 - Accessing the cluster](../exercise-1/README.md)
