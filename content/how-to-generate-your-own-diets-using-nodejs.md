---
title: "Tutorial - Part 1: How to generate your own diets using NodeJS"
include_footer: true 
---


### Introduction

In this part, we will cover how to create a simple command line utility that generates diets. The program will be written in NodeJS. Let’s start!

### Prerequisites
For this tutorial, we are going to use NodeJS and npm. [Here](https://nodejs.dev/learn/how-to-install-nodejs) you can find a tutorial on how to install them.

### Get API key

Before we start, you will need to obtain an ***apiKey*** in order to be able to use our product. To do this, first you need to enter [this link](https://rapidapi.com/genez-io-genez-io-default/api/bespoke-diet-generator/). Once you have done this, you will see the main page and documentation of our API. Click on the **Pricing** section of the page. For this tutorial, you can select the **Basic** version of Subscription. Once you have succesfully subscribed, click on the **API Documentation** button, which will redirect you to the main page, on the *Endpoints* section.
<br><br>


![alt omage](/images/tutorial_pic.jpeg)

Here, in the **red rectangles**, you will find your **apiKey**.

### Setup
* Initialize a new project using ***npm init***. Set the name of the project “DietGenerator” and all the other fields are optional and can be skipped by pressing Enter (however, for a production application, it is advisable to set all the fields accordingly).
* Install axios library for network requests using ***npm install axios***.
* Create a new file index.js and write the following content inside it.
<details>
<summary style="font-weight: bold; cursor: pointer;">Code</summary>

```Javascript
const axios = require("axios");
	
	// Step 0
	const apiKey = "TODO: Replace with your API key";

	(async () => {
		// Step 1
		const args = process.argv.slice(2);

		// Step 2
		if (args.length === 0) {
			console.error("Wrong arguments!");
			return;
		}

		// Step 3
		const command = args[0];

		if (command === "helloworld") {
			console.log("helloworld")
		}
		// TODO
		// else if {}

	})()
```
</details>
<br>

Firstly, in order to be able to make the requests later on, you need to write your API key in the variable ***apiKey***. At step 1, we are reading the arguments passed via the command line. At step 2, we are checking if the user has passed any arguments and if that’s not the case, we show an error and exit the process. Lastly, at step 3, we are getting the command and process it. When we receive this very simple command, we will only print *helloworld* at the standard output. The *TODO*  comment will be replaced with the handling logic of the other commands.

* Now let’s run the application and make sure that everything is fine! Run the application using the command ***node index.js helloworld***. You should see a “helloworld” string printed at the standard output.

<br>

### Diet Generation
Instead of getting countless recipes or broad meal templates, our algorithm generates uniquely specific meal plans that are easy to follow. The biometrics and the personal preferences of the user are taken into consideration when a diet is generated. Having that said, before we generate a diet, we need to create a profile for a user.

Let’s create the command for generating a new user. We can generate a new user in CraveAPI using the **PUT /user** HTTP call.

Now we can replace the *TODO* comment in our code with the handler for the new *createUser* command:
<details>
<summary style="font-weight: bold; cursor: pointer;">Code</summary>

```Javascript
else if (command === "createUser") {
		// Step 1
		const [height, weight, dateOfBirth, sex, activityLevel] = args.slice(1)
		const options = {
			method: 'POST',
			url: 'https://crave-api.p.rapidapi.com/user',
			headers: {
				'content-type': 'application/json',
				// Step 2
				'X-RapidAPI-Key': apiKey,
				'X-RapidAPI-Host': 'crave-api.p.rapidapi.com'
			},
			// Step 3
			data: JSON.stringify({
				"height": parseInt(height),
				"weight": parseInt(weight),
				"dateOfBirth": dateOfBirth,
				"sex": sex,
				"activityLevel": activityLevel
			})
		};

		// Step 4
		const response = await axios.request(options).catch((error) => console.log(error.response));
		console.log("User created Successfully! User ID is:", response.data.id);
  }
```
</details>
<br>
Let’s understand what is happening in this code snippet:

* Step 1: We take the parameters that are passed when our command line utility is run.
* Step 2: We set the API Key from RapidAPI. Make sure you have set the value of the **apiKey** variable correctly.
* Step 3: Put the parameters in the body request.
* Step 4: Make the request and display the user id to the standard output.

Now that we have our user, we can start generating the diet now. We have to decide on three parameters: how many kilograms does the user want to lose, what is the number of days for which the diet should be generated and for what user is this diet for. For this, we have to create a new command that will be named *createDiet* and that receives three parameters *weightGoal*, *dietDuration* and *userId*. This command will generate and display the diet. Let’s modify the code to incorporate this new command logic:

<details>
<summary style="font-weight: bold; cursor: pointer;">Code</summary>

```Javascript
else if (command === "generateDiet") {
		// Step 1
		const [userId, dietType, weightGoal, dietDuration] = args.slice(1)
		const options = {
			method: 'PUT',
			// Step 2
			url: 'https://crave-api.p.rapidapi.com/user/'+ userId +'/diet',
			headers: {
				'content-type': 'application/json',
				'X-RapidAPI-Key': apiKey,
				'X-RapidAPI-Host': 'crave-api.p.rapidapi.com'
			},
			data: JSON.stringify({
				"dietType": dietType,
				"weightGoal": weightGoal,
				"dietDuration": dietDuration
			}),
		};

		const response = await axios.request(options)
			.catch(function (error) {
				console.error(error);
			});

		// Step 3
		response.data.dailyPlan.forEach((plan, index) => {
			console.log(`Day ${index + 1}:`)
			for (const meal of plan.meals) {
				console.log(`   For meal ${meal.type} you should eat:`);
				for (const ingredient of meal.ingredients) {
					console.log(`   ${ingredient.quantity} grams of ${ingredient.name} with id ${ingredient.id}`);
				}
				console.log("");
			}
		});
	}
```
</details>
<br>

* Step 1: Same as before, we parse the arguments from the command line command.
* Step 2: We compose the URL which contains the argument userId.
* Step 3: We iterate through the response and display the diet to the standard output.

Now let’s run the code!

### How to test it
Let’s test the whole flow. First we have to create the user. The user will be a woman born in 1991 that has 170 cm height, 80 kg and is very active:

> node index.js createUser 170 80 1991-03-03 FEMALE VERY_ACTIVE

    User created Successfully! User ID is: 62af3a0fc1767a00de5b03cf

Then we create the diet for her for one week (7 days), taking into consideration that she wants to lose 10 kg (from 80 kg to 70 kg):

> node index.js generateDiet 62af3a0fc1767a00de5b03cf MEDITERRANEAN 70 7

    Day 1:
		 For meal BREAKFAST you should eat:
		 150 grams of Garden salad blend  with id 616eca7d953bf31b472e0ec2
		 150 grams of Cucumbers with id 616eca68953bf31b472e0e7c
		 105 grams of Kale leaves with id 616f8feb17631b48b113df28
		 109 grams of Fish, Tuna, canned in brine, solids with id 616eca94953bf31b472e0f16
		 14 grams of Seeds, Walnuts, raw with id 616eca95953bf31b472e0f1a

		 For meal SNACK1 you should eat:
		 86 grams of Blackberries, raw with id 616eca5b953bf31b472e0e4f
		 70 grams of Tomatoes,grape, ripe with id 616eca63953bf31b472e0e6a
		 90 grams of Chicken, pastrami deli with id 616eca65953bf31b472e0e71

		 For meal LUNCH you should eat:
		 78 grams of Pomegranate, peeled with id 616eca84953bf31b472e0edd
		 144 grams of Broccoli, raw with id 616eca5d953bf31b472e0e57
		 121 grams of Chicken, drumsticks, meat and skin, boneless with id 616eca64953bf31b472e0e6e

		 For meal SNACK2 you should eat:
		 176 grams of Radishes with id 616eca86953bf31b472e0ee4
		 142 grams of Egg, grade A, whole with id 616eca64953bf31b472e0e6f

		 For meal DINNER you should eat:
		 160 grams of Tomatoes with id 616eca93953bf31b472e0f14
		 174 grams of Fish, Halibut Fillet with id 616eca94953bf31b472e0f15
		 11 grams of Oil,Vegetable oil with id 616eca92953bf31b472e0f0e

	Day 2:
		 For meal BREAKFAST you should eat:
		 150 grams of Green bell pepper with id 616eca74953bf31b472e0ea6
		 117 grams of Red pepper with id 616eca8c953bf31b472e0ef8
		 180 grams of Liquid egg whites with id 616fac7017631b48b113df2e
		 33 grams of Cheese, Colby Jack with id 616eca5f953bf31b472e0e5d

		 For meal SNACK1 you should eat:
		 57 grams of Blueberries, raw with id 616eca5b953bf31b472e0e50
		 88 grams of Green bell pepper with id 616eca74953bf31b472e0ea6
		 68 grams of Fish, Tuna, canned in brine, solids with id 616eca94953bf31b472e0f16
		 12 grams of Seeds, Walnuts, raw with id 616eca95953bf31b472e0f1a

		 For meal LUNCH you should eat:
		 191 grams of Watermelon with id 616eca95953bf31b472e0f1b
		 200 grams of Radishes with id 616eca86953bf31b472e0ee4
		 52 grams of Cauliflower, raw with id 616eca62953bf31b472e0e66
		 147 grams of Fish, Flounder fillet with id 616eca89953bf31b472e0eed
		 12 grams of Seeds, Macadamia nuts, raw with id 616eca88953bf31b472e0eeb

		 For meal SNACK2 you should eat:
		 59 grams of Mixed berries, frozen with id 616eca6e953bf31b472e0e94
		 146 grams of Egg, grade A, whole with id 616eca64953bf31b472e0e6f

		 For meal DINNER you should eat:
		 200 grams of Asparagus with id 616f8d3917631b48b113df24
		 120 grams of Fish, Tuna, canned in brine, solids with id 616eca94953bf31b472e0f16
		 16 grams of Oil, Grapeseed oil with id 616eca73953bf31b472e0ea2

     ...

That’s it! Congratulation! You have generated your first diet using CraveAPI!

### Conclusion

We have created a command line tool capable of generating customized diets using the CraveAPI. The code of the command line tool is available here. Have fun and stay healthy!


## [ &raquo; Part 2: How to configure the diet and replace ingredients](/how-to-configure-the-diet-and-replace-ingredients)
