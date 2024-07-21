# Creating Open AI AWS Amplify Application

As an extension to my CloudAPI challenge, I instinctively knew sooner than later I will need to extend the project in AWS Amplify. AWS Amplify is a collection of services for building applications.

Here is the list of services **AWS Amplify** offers

- Authentication
- Storage
- GraphQL API
- DataStore
- REST API
- Analytics
- Push Notifications
- Extended Reality
- PubSub
- AI/ML Predicitions


## Set up AWS Amplify CLI 

1. Install Node.js and NPM `npm install -g @aws-amplify/cli`
2. Configure Amplify CLI with AWS Credentials. This will require you to create an IAM User , Add Permission to the IAM User by attaching policy **AdministratorAccess-Amplify**.
```amplify configure```
3. From the **Security Credentials** tab add **Access Key**.
4. Add the **accessKeyId** and **secretAccessKey** in console.


## Set Up UI Component and Authentication

1. Start by creating a basic react app.

```
npx create-react-app react-authentication 
```
2. Start the Application.

``` 
cd react-authentication
npm start
```
![react-app](../assets/Screenshot%202024-07-21%20at%2009.35.08.png)

## Configuring the Backend for Our React Application

1. Initialize AWS Amplify ``` amplify init```
2. As a result A directory with name "amplify" directory got created with necessary codes.
3. Create an Auth Service ``` amplify add auth ```.

![add auth](../assets/add%20auth.png)
This will add a directory `backend`.
4. Push the authentication configuration to our app ```amplify push```.

## Amplify UI React Components
1. Add the necessary npm package to our application ```npm install aws-amplify @aws-amplify/ui-react```.
2. Modify the index.js file located in the directory ```cd /react-authentication/src``` and call the configure function.
```
import Amplify from 'aws-amplify'
import config from './aws-exports'
Amplify.configure(config)
```
## Integrating Auth Components with React App
