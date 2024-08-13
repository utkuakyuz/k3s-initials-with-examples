To enable nginx load balancer service:

1. Apply `nginx-lb.yaml`
2. Apply `deployment.yaml`
3. Copy the index files into nginx pods:
   ```sh
   kubectl cp index1.html 'nginx-deployment.....:/usr/share/nginx/html/index.html'
   ```
4. Test with:
   ```sh
   while true; do curl http://172.16.10.88/index.html && sleep 1; done;
   ```
