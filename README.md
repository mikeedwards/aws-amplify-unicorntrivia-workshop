# Unicorn Trivia Workshop - Web Version

# Configuring your computer

Before we dive into building the React Web Client, please download a copy of this branch [here](https://github.com/awslabs/aws-amplify-unicorntrivia-workshop/archive/unicorn-trivia-web-workshop.zip)

Once you successfully download a copy and unzip its contents, open up a terminal in that folder to follow along with the instructions below.

Install Node by visiting the official Node.js download page [here](https://nodejs.org/en/download) and selecting the installer for your operating system.

# React Web Walkthrough

## Step 1: Preparing the project

1. Navigate to the root directory of the client folder you downloaded earlier.
1. Run `npm install` from the terminal to install dependencies detailed in `package.json`

## Step 2: Updating the HLS Stream Source in the Video Component

Now that our environment is all set up, we are ready to begin implementing our application! React applications are made up of “Components". Let's begin by creating the Video Player component that will display our video stream in the browser.

Navigate to `./src/components/App/Video/component.js`. The `component.js` file houses the code which defines how we display our livestream on the client device. Our goal for the Video Component is to update the `src` in the `constructor` function of the component. This `src` variable will point to the HLS stream you set up earlier and it will display the stream in your Video Player.

We will now connect the video player to our live streaming backend using the Medistore egress URL generated in the previous step. 
1. Find the `constructor` and replace the value for the `src` key in `this.state` variable to your Mediastore egress URL. If you lost your MediaStore egress url from the amplify livestream setup, you can run `amplify livestream get-info` from the AdminPanel directory to get the MediaStore url. The url should be after the label `MediaStore Output Url:`.
.
	```javascript         
	constructor(props) {
		super(props);
		this.state = {
			src: '#YOUR_MEDIASTORE_URL_HERE'
		};
	}    
	```

## Step 3: Subscribing to the GraphQL API backend
In this section we will be subscribing our client to the back end GraphQL API hosted in AWS AppSync.

1. Our first goal is to pull in one of the files generated by amplify during our AdminPanel set up into the client folder that you chose to build. We will do this by copying the file from the AdminPanel folder you downloaded earlier into your client folder that you downloaded at the beginning of this client walkthrough. We will copy the `aws-exports.js` file from the AdminPanel directory in the AdminPanel folder into the client folder you are currently working out of.
	1. Look for the file `aws-exports.js` in the `./src` folder inside the AdminPanel directory. Copy that over to the `./src` folder inside your client folder.
	1. **Great!** Now all of our files are copied! Let's begin implementing!

**Well Done!** Now we have configured our application code to push and pull data from our GraphQL API. Let's move on to updating our AWS AppSync resolvers and mutations!

## Step 4: Update the backend
1. Open the [AppSync Console](https://console.aws.amazon.com/appsync/home) and navigate to your AppSync endpoint.
1. Once you select your AppSync endpoint on the left side select Schema.
    ![Appsync Schema](.images/Appsync_Schema.png)
1. You now should see your schema that was auto generated for you from Amplify. On the right side you should see a section called Resolvers. Search for `Mutation` in the text box and then select the clickable link next to `updateAnswer(...):Answer`
    ![Appsync Resolver](.images/Appsync_Resolvers.png)
1. You are now presented with a Request Mapping Template and a Response Mapping Template.
    1. We are going to change the Request Mapping Templateto do the appending of the array.
    1. Navigate/search for `#set( $expression = "SET" )` and look for this line (should be near line 42):
        ```vtl
        #set( $expression = "$expression $entry.key = $entry.value" )
        ```
    1. Replace this line with:
        ```vtl
        #if ($util.matches($entry.key, "#answer"))
            #set( $expression = "$expression $entry.key = list_append(if_not_exists($entry.key, :empty_list), $entry.value)" )
            $util.qr($expValues.put(":empty_list", $util.dynamodb.toDynamoDB([])))
        #else
            #set( $expression = "$expression $entry.key = $entry.value" )
        #end
        ```
        This checks to see if the field being set is the answer array. If it is the array then it will append the value. We also do a check to see if the field exists and if it doesn't we create an empty array to append our first value to.
    1. Save the resolver in the top right corner.
1. Run the app again and now you should observe the answers are being correctly appended to the array.

## Step 5: Running the application!

Now that we have every section of the application implemented, it's time to run the app in our emulator.

1. From here run the command `npm start` to launch the application in the browser. The browser window should automatically open up. If not, navigate to `http://localhost:3000` to view the application.
1. From your AdminPanel, you can play along by posting a question, answering the question from the app, and then posting the answer to show you whether you got it correct.
1. If you closed the terminal window running the admin panel no problem! Just open a new terminal window and navigate to the admin panel root directory and run `npm start`. Your default browser should now open up the admin panel on localhost:3000!

**Congratulations!** You have now successfully implemented a UnicornTrivia application on one of three suported platforms! 
Now Try sending some questions and answers using the admin panel we configured previously.

Below are some additional resources for further development! Feel free to skip on forward to the clean up section [here](https://github.com/awslabs/aws-amplify-unicorntrivia-workshop#wrap-up)!

## Troubleshooting Notes

1) Refresh your browser window manually or re-run `npm start`.
