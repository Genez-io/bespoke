---
title: "Tutorial - Part 3: How to handle errors"
include_footer: true 
---


### Introduction

Sometimes APIs can fail because of multiple reasons. One of them is that the input that we pass to the API is invalid. With CraveAPI we return errors in a standardize way such that it is easy to see what the reason of the failure. Let’s see how errors can be handled with grace. We will show how to handle the errors on our command line utility that we have built in the previous tutorials.

### Handling errors

Let’s say we want to create a user but we don’t pass anything to the body. Try to run this command:

> node index.js createUser

        {
            error: { status: 'BAD_REQUEST', message: 'dateOfBirth field is required.' }
        }

The output contains an **error** object that has two properties: **status** and **message**. The status code of the HTTP response is 400. There are two indicators that something wrong happened with your request: if the error object exists in the response’s body or if the HTTP response status code is 3XX or 4XX or 5XX. The message field describes what is wrong with your request. In this example, the problem is that the **dateOfBirth** field is missing.

So, let’s handle the errors in our command line utility. We are using axios and whenever the HTTP response’s status code is different than 2XX, an exception will be thrown and it will be handled by the catch block. What we can do in this **catch** block is to prind the error’s message and then exit the program.

<details>
<summary style="font-weight: bold; cursor: pointer;">Code</summary>

```Javascript

const response = await axios.request(options).catch(function (error) {
		 console.log(error.response.data.error.message);
		 process.exit(1);
	});

```
</details>
<br>

We log the error message and then we exit the process with a status value of 1. You can do this for all requests.

### Conclusion

Handling errors is always important. CraveAPI reports API errors in a consistent way and it is always easy to see what is the root cause. You can check the new version of the command line with the errors handled [here](https://github.com/SimpleCapitalTech/CraveAPI-CommandLine/blob/master/part3/index.js). This is the last part of this tutorial! You can check the [Github page](https://github.com/SimpleCapitalTech/CraveAPI-CommandLine) for more information.

## [ &laquo; Part 2: How to configure the diet and replace ingredients](/how-to-configure-the-diet-and-replace-ingredients)