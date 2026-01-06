# Swagger UI for SysML v2 API

Kubernetes manifests to host a lightweight Swagger UI that serves a fetched OpenAPI document and injects an Authorization header for interactive requests.

## Files
- `swagger-flexo-ui-yaml/swagger-ui.yaml`: Deployment + Service (namespace `swagger` by default), init-container fetches and patches the OpenAPI doc.
- `swagger-flexo-ui-yaml/swagger-configmap.yaml`: Swagger initializer that sets the `Authorization` header.
- `swagger-flexo-ui-yaml/swagger-mapping.yaml`: Ambassador/Emissary mappings for `/doc/` and `/spec/`.

## Required placeholders
Replace these values before applying the manifests:
- `OPENAPI_URL` (in `swagger-ui.yaml`): HTTPS URL to the OpenAPI JSON you want to serve.
- `BEARER_TOKEN` (in `swagger-configmap.yaml`): Bearer token to prefill for Swagger requests, or remove the interceptor if you do not want a default token.
- `SWAGGER_HOST` (in `swagger-mapping.yaml`): External host Ambassador should match for `/doc/` and `/spec/`.

Optionally adjust:
- Namespace (`swagger`) in all three YAMLs if you want a different namespace.
- Image tags or resource limits as needed.

## Usage
1) Update the placeholders above.
2) Apply the manifests (order not critical):  
   ```
   kubectl apply -f swagger-flexo-ui-yaml/
   ```
3) Access Swagger UI at `https://<CHANGE_ME_SWAGGER_HOST>/doc/` once the deployment is ready.
