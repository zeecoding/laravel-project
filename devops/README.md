# Row 2 – GitHub + Ansible + Kubernetes (Laravel three-tier)

| Tier | Runs as | Reachable from |
|---|---|---|
| Presentation + Application | `laravel-app` Deployment (nginx + PHP-FPM, code cloned from GitHub) | Internet via NodePort 30080 |
| Database | `mysql` StatefulSet + 2Gi PersistentVolume | Inside the cluster only (ClusterIP) |

## Run (on the EC2 instance)
    sudo apt update && sudo apt install -y ansible git
    git clone https://github.com/zeecoding/laravel-project.git
    cd laravel-project/devops/ansible
    ansible-playbook site.yml          # prepare + deploy + verify
    # or step by step:
    ansible-playbook 01-prepare.yml
    ansible-playbook 02-deploy.yml
    ansible-playbook 03-verify.yml

## Evidence commands for screenshots
    kubectl get nodes -o wide
    kubectl -n laravel get pods,svc,pvc,statefulset,deployment -o wide
    kubectl -n laravel describe svc laravel-app
    kubectl -n laravel logs deploy/laravel-app -c composer-and-migrate
    kubectl -n laravel exec mysql-0 -- sh -c 'mysql -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE" -e "SHOW TABLES;"'
    curl -I http://localhost:30080/

## Redeploy after a git push
    kubectl -n laravel rollout restart deployment/laravel-app
## Scale demo
    kubectl -n laravel scale deployment/laravel-app --replicas=2
## Troubleshooting
    kubectl -n laravel describe pod -l app=laravel-app
    kubectl -n laravel logs deploy/laravel-app -c git-clone
    kubectl -n laravel logs deploy/laravel-app -c composer-and-migrate
    kubectl -n laravel logs deploy/laravel-app -c laravel
