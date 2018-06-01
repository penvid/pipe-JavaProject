pipeline{
  agent none

  environment{
      MAJOR_VERSION = 1
  }
 
  options{
     buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
  }

  
  stages{
       
      stage('Say Hello'){
        agent any

        steps{
             sayHello 'Awesome student'    
        }
      }

     stage('Git Info'){
       agent any

       steps{
         echo "My branch name: ${env.BRANCH_NAME}"
         
         script{
             def myLib = new vidacademy.git.gitStuff();
             echo "My commit: ${myLib.gitCommit("${env.WORKSPACE}/.git")} "
         }
      
      }
     } 
 
      stage('Unit Test'){
          agent{
              label 'apache'
          }
          steps{
           sh 'ant -f test.xml -v'
           junit 'reports/result.xml'
          }
      }
      stage('build'){
         agent{
              label 'apache'
          }
 
         steps{
           sh 'ant -f build.xml -v'
         }

         post{
           success{
              archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
           }
         }

      }

     stage('deploy'){

        agent{
              label 'apache'
          }

        steps{
            sh "if ![ -d  '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ]; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi"
            sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
        }
     }

     stage("Running on CentOS"){
         agent {
           label 'CentOS'
         }
        steps{
          sh "wget http://vidyas3.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
          sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
        }
     } 

    stage("Test on Debian"){
         agent{
            docker 'openjdk:8u151-jre'  
         }
         steps{
             sh "wget http://vidyas3.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
             sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
         }
     }

    stage("Promote to green"){
        agent {
               label 'apache'
          }
        when{
           branch 'master'
        } 
        steps{
             sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar" 
        }
 
    }
   
    stage('Promote to master'){
        agent{
            label 'apache'
        }

       when{
           branch 'development'
       }
       steps{
            echo 'Stashing any local cahges'
            sh 'git stash'
            echo 'checking out development branch'
            sh 'git checkout development'
            echo 'checking out master'
            sh 'git pull origin'
            sh 'git checkout master'
            echo 'merging dev into master'
            sh 'git merge development'
            echo 'pushing to origin master'
            sh 'git push origin master'  
            echo 'Tagging the release'
            sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
            sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
       }
    }
}
   post{
     success{
        emailext(
             subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] promoted to master",
             body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' promoted to master":</p>
             <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
             to: "vidya_suresh@yahoo.com"
        ) 
     }
   }

}
