# kube-vue

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Test it in a K8s environment
```
kubectl proxy --www=dist 
```
open http://localhost:8001/static
