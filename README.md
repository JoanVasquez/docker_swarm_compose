version: 3
services:
  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10
      restart_policy:
        condiction: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: any_image
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condiction: on-failure

  result:
    image: any_image
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 3
        delay: 10s
      restart_policy:
        condiction: on-failure
  
  worker:
    image: any_image
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condiction: on-failure
        deploy: 10s
        max-attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]
        
 docker stack deploy -c [yamel file name].yml [app name]
 docker slack services [app name]
 docker slack ps [app name]
