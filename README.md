kube-mastodon
-------------

This repository contains everything you need to get a Mastodon server running on Kubernetes.

Copy the template files to their proper places:

```
cp deploy/ingress.yml.template deploy/ingress.yml
cp deploy/secret.yml.template deploy/secret.yml
```

Now fill in the proper values for those files.

Then deploy everything else:

```
kubectl create -f deploy/config.yml
kubectl create -f deploy/secret.yml

kubectl create -f deploy/postgres-volume-claim.yml
kubectl create -f deploy/postgres-deployment.yml
kubectl create -f deploy/postgres-service.yml

kubectl create -f deploy/redis-volume-claim.yml
kubectl create -f deploy/redis-deployment.yml
kubectl create -f deploy/redis-service.yml

kubectl create -f deploy/web-deployment.yml
kubectl create -f deploy/web-service.yml

kubectl create -f deploy/streaming-deployment.yml
kubectl create -f deploy/streaming-service.yml

kubectl create -f deploy/sidekiq-deployment.yml

kubectl create -f deploy/ingress.yml
kubectl create -f deploy/external-dns.yml
```

Now look up the pod name for the web container:

```
kubectl get pods
```

Finally, connect to the web pod so we can set up the database
(use the pod name from before):

```
kubectl exec mastodon-web-c6cc65857-bhqlg  -i -t -- bash -il
```

Once you're connected, run this command to set up the db,
but don't disconnect.

```
$ RAILS_ENV=production bundle exec rails db:migrate
```

Now in the same session, generate VAPID keys:
```
$ RAILS_ENV=production bundle exec rake mastodon:webpush:generate_vapid_key
```

Finally update your secret with the VAPID keys, and replace the secret:
```
kubectl replace -f deploy/secret.yml
```

You may need to kill your streaming, sidekiq, and web pods to get the new env vars.