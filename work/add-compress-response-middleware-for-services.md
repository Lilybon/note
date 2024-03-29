# Add Compress Response Middleware for Services

After sharing [Reduce Initial Bundle Size for CMS](https://hackmd.io/@lilybon/reduce-initial-bundle-size-for-cms) with my teammates, my team lead finds out the way to add compress response middleware for gzip content encoding. In this method, we don't need to add [CompressionWrbpackPlugin](https://webpack.js.org/plugins/compression-webpack-plugin/) to generate `.gz` file or add [compression module](https://expressjs.com/zh-tw/advanced/best-practice-performance.html) in our express server. Just add a configuration for [Traefik middleware](https://doc.traefik.io/traefik/middlewares/overview/). Here are simple steps below.

### 1. Add Middleware

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: compress-response
spec:
  compress: {}
```

### 2. Apply Middleware for K8S Ingress

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: admin-ingress
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`SLD.TLD`)
      kind: Rule
      services:
        - name: admin
          port: xxxx
      middlewares:
        - name: max-request-body-size-limit
    - match: Host(`SLD.TLD`) && PathPrefix(`/page`)
      kind: Rule
      services:
        - name: admin-vue
          port: yyyy
      middlewares:
        - name: compress-response
```

### 3. Final Result

#### AS-IS

User pays about 1~4 seconds for the page response.

![network before](https://i.imgur.com/j7IQsLd.png)

#### TO-BE

User pays about 0~2 seconds for the page response.

![network after](https://i.imgur.com/xrecMiZ.png)

### Reference

- [K8S - Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Traefik - Compress](https://doc.traefik.io/traefik/middlewares/http/compress/)

###### tags: `Work` `K8S`
