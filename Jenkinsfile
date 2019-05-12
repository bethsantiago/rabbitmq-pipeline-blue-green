/*
RABBITMQ Pipieline for Deploy Blue / Green
Objetivos: Realizar deploy de nova versão do RabbitMQ e importar todas as filas e mensagens.
Pré-requisits: Ter RabbitMQ instalado no ambiente Kubernetes
Autor: Elizabeth Santiago / Ivan Santos



*/
pipeline {
	agent {
	    kubernetes {
	    	label 'rabbit-blue-green'
	    	yaml"""
	          apiVersion: v1
	          kind: Pod
	          spec:
	              containers:
	              - name: kubectl
	              	image: lachlanevenson/k8s-kubectl
	              	command: 
	              	- cat
	              	tty: true
	              - name: rabbitmqadmin
	                image: activatedgeek/rabbitmqadmin
	                privileged: true
	                command:
	                - cat
	                tty: true
	                volumeMounts:
	                - mountPath: /backup/
	                  name: rabbitmq-backup
	              volumes:
	              - name: rabbitmq-backup
	                persistentVolumeClaim:
	                  claimName: rabbitmq-backup

	          """
	  	}
	}

	environment {
        BLUE = 'rabbitmq-blue.rabbitmq-deployment.svc.cluster.local'
        GREEN = 'rabbitmq-green.rabbitmq-deployment.svc.cluster.local'
        BACKUP_DIR = '/backup'
    }

  	stages {

	    stage ('Export/Import definitions RabbitMQ') {
	      	steps {
	      		container('rabbitmqadmin') {
		        	scripts {
		        		sh 'rabbitmqadmin --host ${BLUE} export ${BACKUP_DIR}/rabbitmq-blue-definitions.json'

		        		sh 'rabbitmqadmin --host ${GREEN} import ${BACKUP_DIR}/rabbitmq-blue-definitions.json'
		        	}
		        }
	      	}
	    }

	    stage ('Set Federation') {
	    	steps {
	    		container('kubectl') {
		    		scripts {
		    			sh '''
		    				rabbitmqctl \
		    					set_parameter \
		    					federation-upstream blue \
		    					'{"uri":"amqp://${BLUE}"}'
		    			
		    			'''
		    		}
	    		}
	    	}
	    }

	    stage ('Set Policy') {
	    	steps {
	    		container('kubectl') {
		    		scripts {
		    			sh '''
		    				rabbitmqctl \
		    					set_policy blue-green-migration \
		    					--apply-to queues ".*" \
		    					'{"federation-upstream":"blue"}'
		    			'''
		    		}
	    		}
	   		}
	   	}

	   	stage ('Set Shovel') {
	   		steps {

	   			scripts {
	   				sh '''
	   					rabbitmqctl set_parameter shovel drain-blue \
	   					'{"src-protocol": "amqp091", \
	   					"src-uri": "amqp://rabbitmq-blue.rabbitmq-deployment.svc.cluster.local", \
	   					"src-queue": "teste-blue-green", \
	   					"dest-protocol": "amqp091", \
	   					"dest-uri": "amqp://rabbitmq-green.rabbitmq-deployment.svc.cluster.local", \
	   					"dest-queue": "teste-blue-green"}'

	   					'''
	   			}
	   		}
	   	}


	}
}
