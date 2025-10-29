# flexo-sysmlv2-swagger - same origin for UI and API
## Option A — Separate swagger-ui Service + Ingress path /doc (most common)
1. Deploy Swagger UI (fetch spec, point servers to / so calls hit your API root) 
   'swagger-ui.yml'
2. Route /doc on your existing API host to the UI service.
Update your current Ingress for the API host (e.g., flexo.openmbee.org) to add a second path
'ingress-swaggerv2.yml'

Now users open https://flexo.openmbee.org/doc to see Swagger UI.
“Try it out” calls go to https://flexo.openmbee.org/… (same origin, no CORS).

## Option B — Sidecar in your API Deployment (one Service, one Ingress)

Add a second container running Swagger UI inside your existing API Deployment, and expose its port as another Service port or simply route /doc to the same Service/Pod but to the Swagger container’s port via a named port. This keeps everything in one k8s object, but couples UI and API rollouts. If you want this version, I can generate the exact sidecar snippet.

## Apply
kubectl apply -f swagger-ui.yaml
kubectl apply -f ingress-swaggerv2.yaml

# flexo-sysmlv2-swagger - different origin for UI and API
host Swagger UI at https://swaggerv2.openmbee.org/
 and make its “Try it out” calls go to https://flexo.openembee.org
 by patching the OpenAPI spec’s servers array at pod start.


## Important: CORS (because different origins)

Since the UI origin is https://swaggerv2.openmbee.org and the API origin is https://flexo.openmbee.org, your API must allow CORS. If you’re using NGINX Ingress for the API, add something like this on the API’s Ingress:

metadata:
  annotations:
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://swaggerv2.openmbee.org"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, PATCH, DELETE, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-headers: "Authorization,Content-Type,*"
    nginx.ingress.kubernetes.io/cors-allow-credentials: "true"


## Optional: multiple environments in a dropdown

If you want users to switch between prod/staging/local, patch the spec to include several servers:

jq '.servers = [
  {"url": "https://flexo.openmbee.org", "description": "Prod"},
  {"url": "https://flexo-staging.openmbee.org", "description": "Staging"}
]' /spec/openapi.json > /spec/openapi.patched.json


## Alternative: configure via swagger-initializer.js

If you prefer not to fetch/patch the spec, you can ship a custom swagger-initializer.js (via a ConfigMap) that tells Swagger UI to use the raw spec URL and then rely on the spec’s own servers list:

apiVersion: v1
kind: ConfigMap
metadata:
  name: swagger-ui-config
data:
  swagger-initializer.js: |
    window.ui = SwaggerUIBundle({
      url: "https://raw.githubusercontent.com/Open-MBEE/flexo-mms-sysmlv2/develop/resources/openapi.json",
      dom_id: '#swagger-ui',
      validatorUrl: "none",
      deepLinking: true,
      presets: [SwaggerUIBundle.presets.apis, SwaggerUIStandalonePreset],
      layout: "StandaloneLayout"
    });


Mount it over /usr/share/nginx/html/swagger-initializer.js. (You’d still need CORS on the API since the UI will call flexo.openmbee.org)

## Apply
kubectl apply -f swagger-ui-different.yaml
kubectl apply -f ingress-swaggerv2-different.yaml
