# Makefile to manage Trino cluster deployment

# VARIABLES
# -----------------------------------------------------------------------------
#

# Create docker network if not exists
# -----------------------------------------------------------------------------
.PHONY: create-network
create-network:
	@docker network inspect data-network > /dev/null 2>&1 || docker network create --driver bridge data-network
	@echo "Network data-network created"

# compose up with all services
# -----------------------------------------------------------------------------
.PHONY: compose-up-all
compose-up-all: create-network
	@docker compose --profile trino --env-file .env up -d
	@docker ps

# compose up with certbot
# -----------------------------------------------------------------------------
.PHONY: compose-up-certbot
compose-up-certbot: create-network
	@chmod +x ./certbot/docker-entrypoint.sh
	@docker compose --env-file .env --profile certbot up -d
	@docker ps

# get certbot certificate
# -----------------------------------------------------------------------------
DOMAIN=trino.datagang.systems
EMAIL=thanhenc@gmail.com
.PHONY: get-cert
get-cert:
	@docker exec -it certbot certbot certonly --noninteractive --webroot --webroot-path=/var/www/html --email $(EMAIL) --agree-tos --no-eff-email -d $(DOMAIN) --force-renewal

# compose up with 1 trino workers
# -----------------------------------------------------------------------------
.PHONY: compose-up-trino
compose-up-trino: create-network
	@docker compose --profile trino up -d --scale trino-worker=1
	@docker ps

# compose down then remove network
# -----------------------------------------------------------------------------
.PHONY: compose-down
compose-down:
	@docker compose --profile trino --profile spark --profile certbot down
	@docker network rm data-network
	@echo "Network data-network removed"

# Initialize adventureworks database
.PHONY: init-db
init-db:
	@cd ../postgres-data && chmod +x adw14.sh && ./adw14.sh
	@cd ../docker

# Clean temp data
# -----------------------------------------------------------------------------
.PHONY: clean
clean: compose-down
	@rm -r volumes

# Clean Adw14 data
# -----------------------------------------------------------------------------
.PHONY: clean-postgres-data
clean-adw14:
	@rm -r ../postgres-data/data
	@rm -r ../postgres-data/AdventureWorks-oltp-install-script.zip

# Clean all
# -----------------------------------------------------------------------------
.PHONY: clean-all
clean-all: clean clean-postgres-data
	@docker rmi $(docker images -q)
	@docker system prune -f

# Help
# -----------------------------------------------------------------------------
.PHONY: help
help:
	@echo "Usage: make [target]"
	@echo ""
	@echo "Targets:"
	@echo "  create-network    		Create docker network if not exists"
	@echo "  compose-up-all    		Compose up with all services"
	@echo "  compose-up-spark1 		Compose up with 1 spark worker"
	@echo "  compose-up-spark2 		Compose up with 2 spark workers"
	@echo "  compose-up-spark3 		Compose up with 3 spark workers"
	@echo "  compose-up-certbot		Compose up with certbot"
	@echo "  compose-up-trino  		Compose up with 1 trino workers"
	@echo "  compose-down      		Compose down all services then remove network"
	@echo "  get-cert          		Get certbot certificate"
	@echo "  spark-submit      		Spark master submit"
	@echo "  backup            		Backup OLTP, Minio and Hive metastore data"
	@echo "  init-db           		Initialize adventureworks database"
	@echo "  clean             		Clean temp data"
	@echo "  clean-all        		Clean all"
	@echo "  help              		Show this help message"
	@echo ""