# k8s-dd-cnhh

Some useful commands as we run through the hands-on portion:

## Helm

### For those in Azure Cloud Shell

Helm chart link: dtdg.co/ddhelm

Make a directory to hold our content (it's used below!):
`mkdir -p k8s-yaml-files`

Add the repository:
```bash
helm repo add datadog https://helm.datadoghq.com
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
```

Download a copy of the values.yaml:
`wget https://raw.githubusercontent.com/helm/charts/master/stable/datadog/values.yaml && mv values.yaml k8s-yaml-files/`

1. Export the _API_ key:

`echo $DD_API_KEY`

If blank (Azure cloud shell or what-have-you), get the API from https://app.datadoghq.com/account/settings#api scroll to the section after API for the APP keys, and paste it after the `=` sign in the following example:
`export DD_API_KEY=KEY_FROM_DATADOG`

2. Export the _APP_ key:

`echo DD_APP_KEY`

If blank (Azure cloud shell or what-have-you), get the API from https://app.datadoghq.com/account/settings#api, and paste it after the `=` sign in the following example:
`export DD_APP_KEY=KEY_FROM_DATADOG`

3. Once both have been confirmed, you can then install the agent:
```bash
helm install datadogagent \
 --set datadog.apiKey=$DD_API_KEY \
 --set datadog.appKey=$DD_APP_KEY \
 -f k8s-yaml-files/values.yaml datadog/datadog
 ```
 
Things should now be appearing in Datadog, but we'll also want to ensure that we get the kubernetes metrics as well. To do so, open the values.yaml file we used previously, and around line 190, uncomment `env` and add the following on the lnext lines:
```yaml
 - name: DD_KUBELET_TLS_VERIFY
   value: "false"
```

Then we'll re-apply our helm chart in order to update it: 
```bash
helm upgrade datadogagent \
 --set datadog.apiKey=$DD_API_KEY \
 --set datadog.appKey=$DD_APP_KEY \
 -f k8s-yaml-files/values.yaml datadog/datadog
```

### Let's Break Some Things!
Edit the values.yaml file once again in your editor. At lines 194 ~ 200, we'll set the CPU to 20 millicores and RAM to 32 mibibytes - much lower than they should be, but will help us see what happeneds when resources are low. 

The resulting section should look like this: 

```yaml
 resources: 
   requests:
     cpu: 20m
     memory: 32Mi
   limits:
     cpu: 20m
     memory: 32Mi
```

Run helm again to upgrade: 
```bash
helm upgrade datadogagent \
 --set datadog.apiKey=$DD_API_KEY \
 --set datadog.appKey=$DD_APP_KEY \
 -f k8s-yaml-files/values.yaml datadog/datadog
```

Open the event stream to confirm the problems we'd expect: https://app.datadoghq.com/event/stream
