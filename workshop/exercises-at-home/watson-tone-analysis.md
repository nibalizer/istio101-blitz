# At Home Exercises: Watson Tone Analyzer

### Use Watson Tone Analyzer
Watson Tone Analyzer detects the tone from the words that users enter into the Guestbook app. The tone is converted to the corresponding emoticons.

1. Use `bx target --cf` or `bx target -o ORG -s SPACE` to set the Cloud Foundry org and space where you want to provision the service.

2. Create Watson Tone Analyzer in your account.
    ```console
    ibmcloud service create tone_analyzer lite my-tone-analyzer-service
    ```
3. Create a service key for the service.
    ```console
    ibmcloud service key-create my-tone-analyzer-service myKey
    ```
 
4. Show the service key that you created and note the **password** and **username**.
      ```console
      ibmcloud service key-show my-tone-analyzer-service myKey
      ```
 
5. Open the `analyzer-deployment.yaml` and delete the `#` from the `env var` section to un-comment the username and password fields.
 
6. Add the username and password that you retrieved earlier and save your changes.
 
7. Deploy the analyzer pods and service. The analyzer service talks to Watson Tone Analyzer to help analyze the tone of a message.
   ```console
   kubectl apply -f <(istioctl kube-inject -f analyzer-deployment.yaml)
   kubectl apply -f analyzer-service.yaml
   ```
