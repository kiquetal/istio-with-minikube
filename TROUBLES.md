# Troubleshooting Istio Applications

Here are some useful `istioctl` commands for troubleshooting Istio applications:

1.  **Check the status of your Istio control plane and data plane:**
    ```bash
    istioctl status
    ```

2.  **Analyze the configuration of a specific service or workload:**
    ```bash
    istioctl analyze
    ```

3.  **View the routes and virtual services for a given service:**
    ```bash
    istioctl get virtualservice -o yaml
    istioctl get gateway -o yaml
    ```

4.  **Check the configuration of a specific Envoy proxy (e.g., for `idleTimeout` or `idleSeconds`):**
    ```bash
    istioctl proxy-config routes <POD_NAME>.<NAMESPACE>
    istioctl proxy-config listeners <POD_NAME>.<NAMESPACE>
    istioctl proxy-config listeners <POD_NAME>.<NAMESPACE> -o yaml | grep -A 5 -B 5 "idleTimeout"
    ```
    To specifically find the HTTP idle timeout, use the `listeners` command with `grep` to filter for `idleTimeout` within the `HttpConnectionManager` configuration.
    ```
    Replace `<POD_NAME>` and `<NAMESPACE>` with the actual pod name and namespace of your service.

5.  **Retrieve logs from an Envoy proxy:**
    ```bash
    istioctl logs <POD_NAME> -c istio-proxy
    ```

6.  **Verify the sidecar injection status for a pod:**
    ```bash
    istioctl ps <POD_NAME>
    ```

7.  **Enable debug logging for istio-proxy:**
    ```bash
    istioctl proxy-config log <POD_NAME> -n <NAMESPACE> --level debug
    ```
    This command dynamically changes the log level for the `istio-proxy` in the specified pod to debug. Replace `<POD_NAME>` and `<NAMESPACE>` with the actual pod name and namespace of your service. This change is not persistent across pod restarts.

