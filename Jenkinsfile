pipeline {
  agent any

  environment {
    SERVER_IP = credentials('prod-server-ip')
  }

  stages {
    stage('Setup') {
      steps {
        sh '''
          set -e
          python3 -m venv .venv
          . .venv/bin/activate
          python -m pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          set -e
          . .venv/bin/activate
          pytest -q
        '''
      }
    }

    stage('Package code') {
      steps {
        sh "zip -r myapp.zip ./* -x '*.git*' -x '.venv/*'"
        sh "ls -lart"
      }
    }

    stage('Deploy to Prod') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
          sh '''
  
  scp -i "$MY_SSH_KEY" $SSH_OPTS myapp.zip ${username}@${SERVER_IP}:/home/ec2-user/
  ssh -i "$MY_SSH_KEY" $SSH_OPTS ${username}@${SERVER_IP} << 'EOF'
    
    mkdir -p /home/ec2-user/app
    unzip -o /home/ec2-user/myapp.zip -d /home/ec2-user/app/
    cd /home/ec2-user/app
    if [ ! -d "venv" ]; then python3 -m venv venv; fi
    . venv/bin/activate
    python -m pip install -r requirements.txt
    sudo systemctl restart flaskapp.service
EOF
'''
        }
      }
    }
  }
}
