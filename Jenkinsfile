node {
  withEnv(['SUBDOMAIN=cloudapps.example.com']) {

  stage ("Build")
  echo '*** Build Starting ***'
  openshiftBuild bldCfg: 'cotd', buildName: '', checkForTriggeredDeployments: 'false', commitID: '', namespace: '', showBuildLogs: 'false', verbose: 'false', waitTime: ''
  openshiftVerifyBuild apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', bldCfg: 'cotd', checkForTriggeredDeployments: 'false', namespace: '', verbose: 'false'
  echo '*** Build Complete ***'

  stage ("Deploy and Verify in Development Env")

  echo '*** Deployment Starting ***'
  openshiftDeploy apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: '', verbose: 'false', waitTime: ''
  openshiftVerifyDeployment apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: '', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: ''
  echo '*** Deployment Complete ***'

  echo '*** Service Verification Starting ***'
  openshiftVerifyService apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', namespace: 'pipeline-dev', svcName: 'cotd', verbose: 'false'
  echo '*** Service Verification Complete ***'
  openshiftTag(srcStream: 'cotd', srcTag: 'latest', destStream: 'cotd', destTag: 'testready')

  stage ('Deploy and Test in Testing Env')
  echo '*** Deploy testready build in pipeline-test project  ***'
  openshiftDeploy apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: 'pipeline-test', verbose: 'false', waitTime: ''

  openshiftVerifyDeployment apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: 'pipeline-test', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: '10'
  sh 'curl -ks https://cotd-pipeline-test.${SUBDOMAIN}/item.php >/dev/null'

  stage ('Promote and Verify in Production Env')
  openshiftTag(srcStream: 'cotd', srcTag: 'testready', destStream: 'cotd', destTag: 'prodready')
  echo '*** Deploying to Production ***'
  openshiftDeploy apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: 'pipeline-prod', verbose: 'false', waitTime: ''
  openshiftVerifyDeployment apiURL: 'https://openshift.default.svc.cluster.local', authToken: '', depCfg: 'cotd', namespace: 'pipeline-prod', replicaCount: '1', verbose: 'false', verifyReplicaCount: 'false', waitTime: '10'
  sleep 10
  sh 'curl -ks https://cotd-pipeline-prod.${SUBDOMAIN}/item.php >/dev/null'
  }
}
