pipeline {
  agent none

  environment {
    MAJOR_VERSION = 1
  }

 
  stages {
    stage('Say Hello') {
      agent any

      steps {
      	sayHello 'Awesome student'
      }
    }

    stage('Git Information') {
      agent any

      steps {
      	echo "My Branch Name: ${env.BRANCH_NAME}"

      	/* In order to execute our new global library function called gitCommit, 
      	we're going to utilise the 'script' directive */
        script {}
          // We instantiate our library first so we can access it
          def myLib = new linuxacademy.git.gitStuff();
          /* We calling our function on our library. We're passing it the dir
          for our current workspace which is the env.WORKSPACE/.git */
          echo "My Commit: ${myLib.gitCommit("{env.WORKSPACE}/.git")}"
      }
    }
      
    stage('Unit Tests') {
      agent {
        label 'apache'
      }

      steps {
        sh 'ant -f test.xml -v'
        junit 'reports/result.xml'
      }
    }

    stage ('build') {
      agent {
        label 'apache'
      }
      steps {
        sh 'ant -f build.xml -v'
      }
      post {
        success {
          archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
        }
      }
    }

    stage ('deploy') {
      agent {
        label 'apache'
      }
      steps {
        sh "if [ ! -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ]; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME};fi "
        sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
      }
    }

    stage ("Running on CentOS") {
      agent {
        label 'CentOS'
      }
      steps {
        sh "wget http://192.168.1.16/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
      }
    }

    stage ("Test on Debian") {
      agent {
        docker 'openjdk:8u121-jre'
      }
      steps {
        sh "wget http://192.168.1.16/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
      }
    }

    stage("Promote to Green") {
      agent {
        label 'apache'
      }
      when {
        branch 'master'
      }
      steps {
        sh "cp /var/www/html/rectangles/all/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
      }
    }

    stage ("Promote Development Branch to Master") {
      agent {
        label 'apache'
      }
      when {
    	branch 'development'	
      }
      steps {
        echo "Stashing Any Local Changes"
    	sh 'git stash'
    	echo "Checking Out Development Branch"
    	sh 'git checkout development'
    	echo "Checking Out Master Branch"
    	sh 'git pull origin'
    	sh 'git checkout master'
    	echo "Merging Development into Master Branch"
    	sh 'git merge development'
    	echo "Pushing to Origin Master"
    	sh 'git config user.name dhg2;git push origin master'
      }
      post {
        success {
          emailext (
          subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development Promoted to Master",
          body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Development Promoted to Master":</p>
         <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [$env.BUILD_NUMBER}]</a>&QUOT;</p>""",
          to: "dean.hegazi@auspost.com.au"
          )
        }
      }
    }  
  }
  post {
    failure {
      emailext(
        subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
        body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed!":</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [$env.BUILD_NUMBER}]</a>&QUOT;</p>""",
        to: "dean.hegazi@auspost.com.au"
      )
    }
  }  
}