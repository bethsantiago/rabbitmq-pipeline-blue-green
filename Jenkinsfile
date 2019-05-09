pipeline {
  agent {
    label 'kubernetes'
    yaml"""
          apiVersion: v1
          kind: Pod
          spec:
              containers:
              - name: rabbitmqadmin
                image: activatedgeek/rabbitmqadmin
                privileged: true
                command:
                - cat
                tty: true
                volumeMounts:
                - mountPath: /home/docker/volumes/rabbit-backup/
                  name: rabbitmq-backup
              volumes:
              - name: rabbitmq-backup
                persistentVolumeClaim:
                  claimName: rabbitmq-backup

          """
  }

  	stages {

	    stage ('Export/Import definitions Rabbit') {
	      	steps {
	        	

	      	}
	    }

	    stage ('Set Federation') {
	    	steps {


	    	}
	    }

	    stage ('Set Policy') {
	    	steps {

	 
	   		}
	   	}

	   	stage ('Set Shovel') {
	   		steps {


	   		}
	   	}

	}
}
