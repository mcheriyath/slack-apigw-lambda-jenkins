# slack-apigw-lambda-jenkins
Slash command to build jenkins job via apigw and lambda

1. Need a plugin called [Build Authorization Token](https://plugins.jenkins.io/build-token-root) installed on jenkins <br>
2. Create a [Build token](https://stackoverflow.com/questions/42817169/jenkins-trigger-builds-remotely-authentication-token-option-missing) for the specific job <br>
3. [Create an API gw with a POST resource](https://docs.aws.amazon.com/apigateway/latest/developerguide/integrating-api-with-aws-services-lambda.html#api-as-lambda-proxy-create-api-resources) that uses this body mapping template for Content-Type application/x-www-form-urlencoded :
```
{
    "data": {
        #foreach( $token in $input.path('$').split('&') )
            #set( $keyVal = $token.split('=') )
            #set( $keyValSize = $keyVal.size() )
            #if( $keyValSize >= 1 )
                #set( $key = $util.urlDecode($keyVal[0]) )
                #if( $keyValSize >= 2 )
                    #set( $val = $util.urlDecode($keyVal[1]) )
                #else
                    #set( $val = '' )
                #end
                "$key": "$val"#if($foreach.hasNext),#end
            #end
        #end
    }
}
```
4. Create a lambda function named slack-jenkins with the code from this repo <br>
5. Use the following env variables for the lambda function:
- `JENKINS_TOKEN`: Jenkins token for the remote build
- `JENKINS_URL`: Base URL for your jenkins install (http://FQDNorIP)
- `SLACK_COMMAND`: Name of the Slack slash-command, e.g. /rollout
- `SLACK_TOKEN`: Slack token from the slash command <br>
6. [Create a Slack slash-command](https://api.slack.com/tutorials/easy-peasy-slash-commands) that posts to your API gateway POST resource <br>
7. Run your slash-command with parameters /rollout <job> <branch>, the parameter branch will be passed in to the parameterized build on Jenkins side <br>

#### Future features:
- Slack user validation
- Use of global token for all the allowed jobs for a specific user
- more.....

Big thanks to [EonWhite](https://github.com/eonwhite)
