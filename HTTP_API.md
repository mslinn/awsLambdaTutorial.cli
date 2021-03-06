# HTTP API

The HTTP API features a
[quick create](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop.html#http-api-examples.cli.quick-create)
capability that creates:

1. An HTTP API in front of a Lambda.
2. A default catch-all route.
3. A default stage called `$default` that is configured to automatically deploy changes to the API configuration.
   A deployment is a snapshot of your API&lsqou;s configuration that can be associated with a Stage.
   Each Stage has an invoke URL (a.k.a. endpoint), whose behavior is determined by the Stage settings and the deployment attached to the Stage.
   Auto-deploy enabled stages cannot be deployed manually.

If you wish to type along and have not already performed the instructions on the [previous page](LOGGING.md) please do so now.

If you are resuming these instructions in a new shell, load the environment variables from `setEnvVars`:

```script
$ source setEnvVars
```

## Saving Your Work

We will use these environment variables from `setEnvVars`:

```script
AWS_APIG_PATH_PART=demo   # Part of the URL path to invoke the Lambda function
AWS_APIG_TARGET_ARN="arn:aws:lambda:$AWS_REGION:$AWS_ACCOUNT_ID:function:$AWS_LAMBDA_NAME"
AWS_APIG_NAME=LambdaHTTP
```

Near the end of this page you will be instructed to save a value for the  `AWS_APIG_HTTP_ID` environment variable so you can return to this project at another time.

The value of `AWS_HTTP_INVOCATION_URL` is computed from values of other environment variables by `setEnvVars`.


## Create an API Gateway HTTP API

1. Call the `create-api` command to create an API with the desired name.
   This command maps the Lambda to the route `/$AWS_APIG_PATH_PART`.
   The `POST` in the route key will only allow `POST` method calls on the specified route.

   ```script
   $ ./capture aws apigatewayv2 create-api \
     --name $AWS_APIG_NAME \
     --protocol-type HTTP \
     --route-key "POST /$AWS_APIG_PATH_PART" \
     --target "$AWS_APIG_TARGET_ARN"
   ```

   If you would like to allow all methods in this route, use the keyword `ANY` instead, like this:

   ```script
   $ ./capture aws apigatewayv2 create-api \
     --name $AWS_APIG_NAME \
     --protocol-type HTTP \
     --route-key "ANY /$AWS_APIG_PATH_PART" \
     --target "$AWS_APIG_TARGET_ARN"
   ```

   Output will be something like:

   ```json
   {
       "ApiEndpoint": "https://y5sy8ty98g.execute-api.$AWS_REGION.amazonaws.com",
       "ApiId": "y5sy8ty98g",
       "ApiKeySelectionExpression": "$request.header.x-api-key",
       "CreatedDate": "2020-10-07T21:32:21Z",
       "Name": "LambdaHTTP",
       "ProtocolType": "HTTP",
       "RouteSelectionExpression": "$request.method $request.path"
   }
   ```

   We have created an HTTP API and the resulting JSON has been saved to `.result.json` by the `capture` script.
   Now we will use the `extract` script to extract the `ApiId` value from the JSON file and save the value for
   `AWS_APIG_HTTP_ID` in `setEnvVars`.

    ```script
    ./extract .ApiId AWS_APIG_HTTP_ID
    ```

   `setEnvVars` already computes the invocation URL and stores it an environment variable called `AWS_HTTP_INVOCATION_URL`, like this:

   ```script
   AWS_HTTP_INVOCATION_URL=$AWS_APIG_HTTP_ID.execute-api.$AWS_REGION.amazonaws.com/$AWS_APIG_PATH_PART/
   ```

   Now re-load from `setEnvVars`:

   ```script
   source setEnvVars
   ```

2. Permissions have not yet been provided for the Lambda to be executed by the API Gateway, so
   attempting to invoke would result in API Gateway receiving an `AccessDenied` error,
   which results in the client receiving `{"message":"Internal Server Error"}`.

   To add permissions to the Lambda to allow the API to invoke it, type the following.
   This incantation permits all HTTP methods; to only permit `POST` method invocation,
   change the second-last token in the `source-arn` from `*` to `POST`.

   ```script
   $ aws lambda add-permission \
     --statement-id lambda_perm \
     --action lambda:InvokeFunction \
     --function-name "$AWS_LAMBDA_ARN" \
     --principal apigateway.amazonaws.com \
     --source-arn "arn:aws:execute-api:$AWS_REGION:$AWS_ACCOUNT_ID:$AWS_APIG_HTTP_ID/*/\$default"
   ```

   Output is something like:

   ```json
   {
     "Statement": {
       "Sid": "lambda_perm",
       "Effect":"Allow",
       "Principal": { "Service": "apigateway.amazonaws.com" },
       "Action": "lambda:InvokeFunction",
       "Resource": "arn:aws:lambda:$AWS_REGION:$AWS_ACCOUNT_ID:function:$AWS_LAMBDA_NAME",
       "Condition": {
         "ArnLike": { "AWS:SourceArn": "arn:aws:execute-api:$AWS_REGION:$AWS_ACCOUNT_ID:$AWS_APIG_HTTP_ID/*/$default" }
       }
     }
   }
   ```

3. Now we can make requests to the API gateway's invocation URL like this:

   ```script
   $ curl --no-progress-meter \
     -d "company='Slate Rock and Gravel Company'" \
     -d "emailAddress=fred@flintstone.com'" \
     -d "firstName=Fred" \
     -d "lastName=Flintstone" \
     -d "notes=None" \
     -d "phone=555-123-4567" \
     -d "title=Slob" \
     $AWS_HTTP_INVOCATION_URL
   ```

    Output is:
    ```json
    {"message":"Internal Server Error"}
    ```


## Next Step
Continue on to [Logging Continued](LOGGING2.md).
