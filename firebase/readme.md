#### How to set up firebase.io

Firebase (https://www.firebase.com/) is an online database which allows communication between servers and apps. 
In order for the features of the app to work, it needs a firebase account to read and send data to and from. 
The existing code uses the database at https://gd-prototype.firebaseio.com but this can be substituted for another with a similar tree structure. 
In order to create a new database, you simply need to create an account at www.firebase.com and select “create new app”

##### Setting up database
For the app to run properly, your firebase tree structure must have three main branches: "parameters", "training weights", and "users." The individual user sub-trees will be added when a new user creates an account through the app's sign-in page. The training weights branch should have two sub-branches: "iteration" and "weights". 

![Firebase Tree Structure](https://github.com/zeehun/CrowdML/blob/master/firebase/firebaseTree.png)

* Training Weights

The iteration is the version number of the weights stored by firebase. Every time the server updates these values, this number will increase. This is used to ensure that the gradient generated by the clients is being applied to the correct weight values by checking the current weight iteration number and making sure it matches the iteration number of the weight used to generate the gradient.

The weights value will contain the actual feature weight array

* Parameters

The Parmeters tree will be automatically created when you first run the server. This will take the information needed by the clients and make it available for them to checkout through the client's event listeners. If you change the parameters, it is important that you change the paramIter as well, ensuring that only clients using the most recent parameters will have their gradients accepted.

![Parameter Tree Structure](https://github.com/zeehun/CrowdML/blob/master/firebase/paramTree.png)

* Users

The Users branch contain the data for each user. This is automatically generated by the client code and does not need to be built here. Here is the diagram of an example user sub-branch:

![User Tree Structure](https://github.com/zeehun/CrowdML/blob/master/firebase/userTree.png)

* UID

The code at the top is the user-ID, automatically generated every time a user creates a new account with their unique email/password. 

* gradIter

The 'gradIter' is a private iteraration count used in the client code to ensure that one gradient is not uploaded before the previous value has been received. This is entirely separate from the weight iterator, and is used by the client alone to check that the latest gradient iteration has been received by Firebase before sending a new one.

* gradientProcessed

gradientProcessed is a boolean value set to false with every gradient sent, and set to true by the server once its values have been read. This ensures that a new gradient value is not sent until the previous one has been read by the server. gradients contains the gradient array sent by the client. 

* weightIter

This is the Training Weight iteration number used to create the sent gradient; checking that this is equal to the weight iteration allows the server and client to ensure that a valid gradient is being applied to the weight. Every time a weight is collected in order to create a new gradient, the weight iteration of that weight is stored along with it. When the gradient is sent to Firebase, the iteration number is sent with it to allow the server to verify that it is being applied to the correct weight values.

##### Firebase checkout system

In order to ensure that every interaction between the clients, the server, and Firebase happens in the correct, a series of checkout systems have been implemented. Firstly, a unique gradIter for every client allows the client to check that a gradient packet has been received by Firebase before sending a second. Similarly, a gradientProcessed boolean set to false with every gradient sent and set to true when processed by the server makes sure that a gradient has been read by the server before the client sends a second. There are two iterators which prevent any outdated gradients from being applied to the weights. The first is weightIter which is taken by the client every time a weight it pulled, and checked by the server after sending to see if it matches the current weightIter before applying the gradient. This value is updated every time the weight is updated. Finally the paramIter is updated by the administrator every time the parameters are changed, and is checked in a similar manner to the weightIter by the server to make sure the gradients being applied have used the correct and current parameters.

![CheckoutFlowchart](https://github.com/zeehun/CrowdML/blob/master/firebase/Crowd-ML-Checkout.png)

##### Setting up authentication

1. Authentication for server and android/iOS clients are explained in their respective readmes
2. In your project console, go to Auth/Sign In Method' and enable Email/Password
3. Go to Database/Rules and paste the following rules settings:
```
{
    "rules": {
          ".read": true,
          ".write": false,
        "users": {
          "$user_id": {
            ".read": "$user_id === auth.uid",
            ".write": "$user_id === auth.uid"
          }
        }
    }
}
```

In order to disable all read/write abilities for the database, simply set these rules to false.

```
{
    "rules": {
          ".read": false,
          ".write": false
    }
}
```
Note: This will not be able to prevent access for anything with administrator privileges such as the server.