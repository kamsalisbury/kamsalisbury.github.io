---
layout: post
title: Simple Stripe and Firebase subscription and authentication
date:   2018-01-10 07:00:00:33
category: javascript
tags: [php,firebase,javascript,Stripe,gateway]
---

In this post, I am going to show you how you can create a subscription using Stripe and authentication using Firebase.

We will create validate user's credit card and if it is valid, we will create their account on Firebase and then subscribe them to a plan on Stripe.

## What is Stripe

Stripe is an online payment processing for internet businesses. Stripe is a suite of payment APIs that powers commerce for businesses of all sizes. Stripe has a simple fee structure similar to PayPal for incoming transactions, currenly 2.9% + 30¢ for pay-as-you-go accounts. They also offter enterprice pricing.

For more information, see <https://stripe.com>

## What is Firebase

Firebase is Google's mobile platform that helps you quickly develop high-quality apps and grow your business. Firebase was developed by  Firebase, Inc. in 2011 and was later acquired by Google in 2014.

For move information, see <https://firebase.google.com>

## Requirements

1. Stripe account <https://stripe.com>
1. Firebase account <https://console.firebase.google.com>
1. JQuery <https://jquery.com>
1. Twitter Bootstrap <https://getbootstrap.com>
1. Composer <https://getcomposer.org>

## Dependencies
1. Stripe-php <https://github.com/stripe/stripe-php>

## Getting started

1. Setup your Stripe account if you have not already done so and retrieve both your public and secret API keys.

1. Set up your Google Firebase account and enable email/password authentication

1. Retrieve your Firebase configuration settings

    ```json
    {
    "apiKey": "your-app-api-key",
    "authDomain": "your-app.firebaseapp.com",
    "databaseURL": "https://your-app.firebaseio.com",
    "projectId": "your-app",
    "storageBucket": "your-app.appspot.com",
    "messagingSenderId": "your-app-message-id"
    }
    ```

1. `composer require stripe/stripe-php`

1. Your HTML page

1. Include JS files (JQuery, Bootstrap and Stripe\)

    `<script src="https://js.stripe.com/v3/"></script>`

1. The complete script. Add each code to its respective file. i.e css to your `style.css`, js to your `script.js` and `index.html`.
    <script src="https://gist.github.com/jgmuchiri/133d185239cca1905805951cbc4839ee.js"></script>

1. Add the following in your `register.php`

    ```php
    if(isset($_POST['stripeToken'])):
        \Stripe\Stripe::setApiKey("your_stripe_sk_key");
            $token = $_POST['stripeToken'];
            $success = 0;
            $error = '';
            try {
                $customer = \Stripe\Customer::create(array(
                    "description" => $this->input->post('name'),
                    "email" => $_POST['email'],
                    "source" => $token
                ));
                $success = 1;
            } catch (\Stripe\Error\Card $e) {
                $error = $e->getMessage();
            } catch (\Stripe\Error\InvalidRequest $e) {
                $error = $e->getMessage();
            } catch (\Stripe\Error\Authentication $e) {
                $error = $e->getMessage();
            } catch (\Stripe\Error\ApiConnection $e) {
                $error = $e->getMessage();
            } catch (\Stripe\Error\Base $e) {
                $error = $e->getMessage();
            } catch (Exception $e) {
                $error = $e->getMessage();
            }

            if ($success == 1) {
                $success = 0;
                try {
                    \Stripe\Subscription::create(array(
                        "customer" => $customer->id,
                        "items" => array(
                            array(
                                "plan" => $this->input->post('plan'),
                            ),
                        )
                    ));
                    $success = 1;
                } catch (\Stripe\Error\InvalidRequest $e) {
                    $error = $e->getMessage();
                } catch (\Stripe\Error\Authentication $e) {
                    $error = $e->getMessage();
                } catch (\Stripe\Error\ApiConnection $e) {
                    $error = $e->getMessage();
                } catch (\Stripe\Error\Base $e) {
                    $error = $e->getMessage();
                } catch (Exception $e) {
                    $error = $e->getMessage();
                }
            }
            if ($success == 1) {
                $type = 'success';
                $msg = 'You are all set.';
            } else {
                $type = 'error';
                $msg = $error;
            }
            echo json_encode(['type' => $type, 'msg' => $msg]);
        endif;
     ```
If you have set up correctly, when you complete and submit the form, you should have a user registered in firebase and a subscription started in Stripe.

Now the next step (not included here) would be to integrate authentication to your application using firebase. For more information, see <https://firebase.google.com/docs/auth/web/start>

If you have comments or need further information, use comments below.