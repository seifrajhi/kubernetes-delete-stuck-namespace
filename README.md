# Troubleshooting: Namespace Stuck in Terminating State

## Symptom
A namespace is stuck in the Terminating state.

## Cause
If a Kubernetes API extension is not available, the resources managed by the extension cannot be deleted. Failure to delete the API extension causes namespace deletion to fail.

## Resolving the Problem
Follow the steps below to resolve the issue:

1. View the namespaces that are stuck in a Terminating state:

```python
kubectl get namespaces
```

2. Find the resources that are not deleted:

```Powershell
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <terminating-namespace>
```


If the above command returns an error message like "unable to retrieve the complete list of server APIs," continue to the next step.

3. Obtain the API descriptions of the stuck resources:
- Run the following command for each API service that was not deleted:
  ```
  kubectl get APIService <version>.<api-resource>
  ```

  For example:
  ```
  kubectl get APIService v1beta1.custom.metrics.k8s.io
  ```

- Get a description of the API service to continue debugging:
  ```
  kubectl describe APIService <version>.<api-resource>
  ```

4. Ensure the issue is resolved by running:
```bash
kubectl get namespace
```


5. If the issue persists, manually delete the terminating namespace:
- View the namespaces that are stuck in the Terminating state:
  ```
  kubectl get namespaces
  ```

- Select a terminating namespace and view its contents to find out the finalizer:
  ```
  kubectl get namespace <terminating-namespace> -o yaml
  ```

- Create a temporary JSON file:
  ```
  kubectl get namespace <terminating-namespace> -o json >tmp.json
  ```

- Edit the `tmp.json` file, remove the `kubernetes` value from the `finalizers` field, and save the file.

- Set a temporary proxy IP and port:
  ```
  kubectl proxy
  ```

- From a new terminal window, make an API call with the temporary proxy IP and port:
  ```
  curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8001/api/v1/namespaces/<terminating-namespace>/finalize
  ```

6. Verify that the terminating namespace is removed:

```python
kubectl get namespaces
```


7. Repeat the steps for other namespaces stuck in the Terminating state.



