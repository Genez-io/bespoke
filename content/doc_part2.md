---
title: Documentation
subtitle: Tutorial
include_footer: true 
---

## Part 2: How to configure the diet and replace ingredients

### Introduction

In the previous tutorial, we have seen how to create an user and how to generate a diet for them. If you haven’t done it, check it out here. Let’s say that we want to configure the diets to be according to the user’s taste. How do we do this? Or maybe the user has some allergies and they would prefer to avoid some ingredients. The good news is that CraveAPI offers support for customizing diets!

### Diet configuration

There are two ways in which you can control how the diet will look like:

1. An API call which you can use to set the ingredients that the user does not want in his diet.

Here we have an example of a request that tells the Crave algorithm that the user does not want an ingredient at all.

<details>
<summary style="font-weight: bold; cursor: pointer;">Code</summary>

```Javascript
const userId = "62ab2382c1767a00de5b03ce";
	const ingredientId = "62ab1050c1767a00de5b03bc";

	const options = {
		method: 'PUT',
		url: 'https://crave-api.p.rapidapi.com/user/${userId}/ingredients/excluded',
		headers: {
			'content-type': 'application/json',
			'accept-language': 'ro',
			'X-RapidAPI-Key': apiKey,
			'X-RapidAPI-Host': 'crave-api.p.rapidapi.com'
		},
		data: {
			"ingredientIds": [ingredientId]
		}
	};

	const response = await axios.request(options);
```
</details>
<br>

The body of the request is a JSON that contains an array with the list of ingredients that are less desirable for the user with id *userId*. In our example, there is only one ingredient. The list of ingredients from which to pick can be retrieved by making the GET call to route */ingredients*.

Let’s also add a command to get the list of ingredients.

<details>
<summary style="font-weight: bold; cursor: pointer;">Code</summary>

```Javascript

	else if (command === "getIngredients") {
			const options = {
				method: 'GET',
				url: 'https://crave-api.p.rapidapi.com/ingredients',
				headers: {
					'accept-language': 'en',
					'X-RapidAPI-Key': apiKey,
					'X-RapidAPI-Host': 'crave-api.p.rapidapi.com'
				}
			};

			axios.request(options).then(function (response) {
				response.data.forEach((foodCategory) => {
					console.log(`${foodCategory.name}:`);
					foodCategory.ingredients.forEach((ingredient) => {
						console.log(` [${ingredient.id}]: ${ingredient.name}`);
					});
				});

			}).catch(function (error) {
				console.error(error);
			});
		}
```
</details>
<br>

This command will list all the ingredients available in the CraveAPI database together with their identifiers.

2. The second option to customize a diet is an API call that can be used to replace an ingredient from an existing diet. However, an ingredient that exists in an existing diet can not be replaced with any other ingredient. First you have to make a call to retrieve the list of ingredients from which you can pick one that will replace the unwanted ingredient.

<details>
<summary style="font-weight: bold; cursor: pointer;">Code</summary>

```Javascript

const options = {
		method: 'GET',
		url: 'https://crave-api.p.rapidapi.com/user/${userId}/diet/${dayIndex}/${mealType}/${ingredientId}/replace',
		headers: {
			'accept-language': 'ro',
			'X-RapidAPI-Key': apiKey,
			'X-RapidAPI-Host': 'crave-api.p.rapidapi.com'
		}
	};

	const response = await axios.request(options);

```
</details>
<br>

You have to pass in the URL the user’s id, the day’s index, the type of meal (e.g.: breakfast, snack, dinner) and the id of the ingredient that we want to replace.

Once we have the id of the ingredients, we can pick one and make the request.

<details>
<summary style="font-weight: bold; cursor: pointer;">Code</summary>

```Javascript

const options = {
		method: 'PUT',
		url: 'https://crave-api.p.rapidapi.com/user/${userId}/diet/${dayIndex}/${mealType}/${toBeReplacedIngredientId}/replace',
		headers: {
			'content-type': 'application/json',
			'X-RapidAPI-Key': apiKey,
			'X-RapidAPI-Host': 'crave-api.p.rapidapi.com'
		},
		data: '{"ingredientId": newIngredientId}'
	};

	axios.request(options).then(function (response) {
		console.log(response.data);
	}).catch(function (error) {
		console.error(error);
	});

```
</details>
<br>

### How to test it

Firstly, we set the ingredients that our users does not want to have in their diet. The user first have to get the list of ingredients and to do this they have to run:

> node index.js getIngredients

	Vegetables:
	 [616eca55953bf31b472e0e39]: Arugula
	 [616eca55953bf31b472e0e3a]: Asian Stir-fry vegetables
	 [616eca56953bf31b472e0e3b]: Mixed grilled vegetables
	 ...

From this list, the user will select the undesirable ingredients.

> node index.js setExcludedIngredients 62af3a0fc1767a00de5b03cf 616eca8c953bf31b472e0ef9 616eca8d953bf31b472e0efe

	The excluded ingredients are:
	 [616eca8c953bf31b472e0ef9]: Cereals, Quinoa, rice and spinach mix, frozen
	 [616eca8d953bf31b472e0efe]: Cereals, Rye flakes 

Now, the diet can be generated.

> node index.js generateDiet 62af3a0fc1767a00de5b03cf 616ec98827b0e601644583df 70 2

	Day 1:
		 For meal BREAKFAST you should eat:
		 140 grams of Kale leaves with id 616f8feb17631b48b113df28
		 150 grams of Cucumbers with id 616eca68953bf31b472e0e7c
		 92 grams of Mixed lettuce salad with id 616eca7d953bf31b472e0ec3
		 104 grams of Fish, Salmon, smoked fillets with id 616eca8f953bf31b472e0f04

		 For meal SNACK1 you should eat:
		 107 grams of Strawberries with id 616eca91953bf31b472e0f0d
		 58 grams of Red bell pepper with id 616eca8a953bf31b472e0ef3
		 68 grams of Fish, Tuna, canned in brine, solids with id 616eca94953bf31b472e0f16
		 12 grams of Seeds, Walnuts, raw with id 616eca95953bf31b472e0f1a

		 For meal LUNCH you should eat:
		 64 grams of Bananas with id 616eca57953bf31b472e0e41
		 224 grams of Romaine lettuce with id 616f8ee517631b48b113df26
		 60 grams of Tomatoes,grape, ripe with id 616eca63953bf31b472e0e6a
		 119 grams of Fish, Tilapia filet with id 616eca7e953bf31b472e0ec8
		 12 grams of Oil, Grapeseed oil with id 616eca73953bf31b472e0ea2

		 For meal SNACK2 you should eat:
		 200 grams of Romaine lettuce with id 616f8ee517631b48b113df26
		 100 grams of Turkey, organic turkey deli with id 616eca80953bf31b472e0ecd
		 28 grams of Cheese, Mozarella with id 616eca7d953bf31b472e0ec4

		 For meal DINNER you should eat:
		 133 grams of Red pepper with id 616eca8c953bf31b472e0ef8
		 173 grams of Chicken thighs, skinless, boneless, fat trimmed with id 616eca65953bf31b472e0e72

	Day 2:
		 For meal BREAKFAST you should eat:
		 72 grams of Baby spinach with id 616eca57953bf31b472e0e40
		 51 grams of Hummus with id 616eca77953bf31b472e0eae
		 102 grams of Fish, Tuna, canned in brine, solids with id 616eca94953bf31b472e0f16

		 For meal SNACK1 you should eat:
		 82 grams of Mixed berries, frozen with id 616eca6e953bf31b472e0e94
		 117 grams of Mixed lettuce salad with id 616eca7d953bf31b472e0ec3
		 61 grams of Fish, Tuna, canned in brine, solids with id 616eca94953bf31b472e0f16
		 12 grams of Seeds, Walnuts, raw with id 616eca95953bf31b472e0f1a

		 For meal LUNCH you should eat:
		 91 grams of Pineapple tidbits, canned, solids, no added sugar with id 616eca83953bf31b472e0ed8
		 62 grams of Arugula with id 616eca55953bf31b472e0e39
		 224 grams of Romaine lettuce with id 616f8ee517631b48b113df26
		 119 grams of Turkey, breast fillet, raw with id 616eca8a953bf31b472e0ef1
		 15 grams of Seeds, Brazil nuts, raw with id 616eca87953bf31b472e0ee7

		 For meal SNACK2 you should eat:
		 150 grams of Cucumbers with id 616eca68953bf31b472e0e7c
		 55 grams of Tomatoes,grape, ripe with id 616eca63953bf31b472e0e6a
		 100 grams of Turkey, organic turkey deli with id 616eca80953bf31b472e0ecd
		 53 grams of Liquid egg whites with id 616fac7017631b48b113df2e
		 9 grams of Peanut butter, no added sugar with id 616eca82953bf31b472e0ed4

		 For meal DINNER you should eat:
		 83 grams of Carrots with id 616eca61953bf31b472e0e65
		 151 grams of Pork, tenderloin with id 616eca85953bf31b472e0ee0
		 13 grams of Seeds, Macadamia nuts, raw with id 616eca88953bf31b472e0eeb

But let’s say that in the breakfast of the first day, the user would want something else instead of the salmon. Let’s see what ingredients can be used instead of salmon:

> node index.js replaceIngredient 62af3a0fc1767a00de5b03cf 0 BREAKFAST 616eca8f953bf31b472e0f04

	Choose an ingredient from the following list:
	 [616eca63953bf31b472e0e6c] Chicken breast, skinless, boneless 166.18556701030928 grams
	 [616eca7f953bf31b472e0ecc] Chicken, Organic chicken deli 132.13114754098362 grams
	 [616eca64953bf31b472e0e6f] Egg, grade A, whole 159.6039603960396 grams
	 [616eca80953bf31b472e0ecd] Turkey, organic turkey deli 183.1818181818182 grams
	 [616fa3ab17631b48b113df2a] Milk alternatives, Dairy-free cheddar cheese substitute 58.405797101449274 grams
	 [616fa3f217631b48b113df2b] Milk alternatives, Dairy free mozzarella cheese substitute 58.405797101449274 grams
	 [616fad4f17631b48b113df2f] Egg substitute, plant based 104 grams
	 [616fac7017631b48b113df2e] Liquid egg whites 403 grams
	 [616faf5717631b48b113df30] Meat substitute, plant based ground 77.5 grams
	 [616fb00717631b48b113df31] Meat substitute, plant based sausage 55.58620689655172 grams
	 [616eca6c953bf31b472e0e8a] Cheese, Feta cheese, regular fat 56.56140350877193 grams
	 [616eca72953bf31b472e0e9e] Cheese, goat cheese, plain 52.16828478964401 grams
	 [616eca72953bf31b472e0e9f] Cheese, goat cheese semi-soft 41.87012987012987 grams
	 [616eca72953bf31b472e0ea0] Cheese, Gorgonzola  47.134502923976605 grams
	 [616eca75953bf31b472e0ea9] Cheese, light cream cheese spread 88.08743169398907 grams
	 [616eca7d953bf31b472e0ec4] Cheese, Mozarella 62.96875 grams
	 [616eca81953bf31b472e0ed1] Cheese, Parmesan 37.68115942028985 grams
	 [616eca8d953bf31b472e0efc] Cheese, Ricotta, part skimmed 118.52941176470588 grams
	 [616eca6b953bf31b472e0e87] Cheese, Swiss cheese 46.32183908045977 grams
	 [616eca93953bf31b472e0f13] Milk alternatives, Tofu, firm 156.50485436893203 grams
	 [616eca73953bf31b472e0ea4] Yogurt, Greek yogurt low fat 265.13157894736844 grams
	 [616eca7a953bf31b472e0ebb] Fish, Salmon, marinated fillet 101.38364779874213 grams
	 [616eca8e953bf31b472e0eff] Fish, Sardines, canned in oil, solids 59.92565055762082 grams
	 [616eca8f953bf31b472e0f04] Fish, Salmon, smoked fillets 104 grams
	 [616eca8f953bf31b472e0f05] Fish, Tuna, smoked 150.37313432835822 grams
	Write the ID of one of the ingredients above:

If the user wants organic turkey instead of salmon, he just needs to type “616eca80953bf31b472e0ecd” and the ingredient will be replaced in the diet.

### Conclusion

It is important to eat what you like in order to mantain a healthy lifestyle. CraveAPI offers the support to customize diets. The code for this tutorial can be found [here](https://github.com/SimpleCapitalTech/CraveAPI-CommandLine/blob/master/part2/index.js).

## [ &laquo; Part 1: How to generate your own diets using NodeJS](/doc_part1)

## [ &raquo; Part 3: How to handle errors](/doc_part3)