Three Tier Architecture Model
-
- Every real time apps are three-tier built. These apps are written using 3 different layers - Frontend (presentation layer), backend (logic layer) and database (data layer)
- When we want to checkout any product online we check on frontend, to get more details we fetch them from database where details are stored. 
  - To display info from DB to UI there has to be backend. So when user performs action. When user clicks on product, details are fetched from DB but here backend co-ordinates with our frontend and DB.
  - Here in backend, developers write their logic for application to perform.

3 Tier Application Overview
- 
- For any application there are 2 types of flows, user flow and admin flow
- Lets say we're developing robot app for now

- User Flow
  - For user accessing any app, he need to login or register
  - Once logged in, user will see catalogue details like AI products or robots
  - After clicking on product, he will see rating for it.
  - Then if he likes product, he can add to cart
  - After which he can proceed for payment
  - Then he can add shipping details
  - At last order will get completed
 
  Login/register - Catalogue - Rating - Cart - Payment - Shipping - Ordder complete

  - Here for each step we need logic to be written. This can be done using single application binary which is monolithic or using separate code for each application which is microservices to make all the components individually deployable.
 
  - Here we'll use microservice architecture.
    - If we add one more category to catalogue section and application is written in monolithic, any changes made might impact other components as well. QA will have to validate all components for single chnange.
    - In microservices, each component will have individual APIs, can be independently deployed. So QA will use specific APIs to test without worrying about rest components.

