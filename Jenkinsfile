properties(
  [
    parameters(
      [
        string(
          defaultValue: 'x86_64,ppc64le',
          description: 'A comma separated list of architectures to run the test on. Valid values include [x86_64, ppc64le, aarch64, s390x].',
          name: 'ARCHES'
        ),
        string(
          defaultValue: 'https://github.com/redhat-multiarch-qe/multiarch-ci-libraries',
          description: 'Repo for shared libraries.',
          name: 'LIBRARIES_REPO'
        ),
        string(
          defaultValue: 'dev-v1.2.1',
          description: 'Git reference to the branch or tag of shared libraries.',
          name: 'LIBRARIES_REF'
        ),
        string(
          defaultValue: '',
          description: 'Repo for tests to run. If left blank, the current repo is assumed (*note* this default will only work for multibranch pipelines).',
          name: 'TEST_REPO'
        ),
        string(
          defaultValue: '',
          description: 'Git reference to the branch or tag of the tests repo.',
          name: 'TEST_REF'
        ),
        string(
          defaultValue: 'tests',
          description: 'Directory containing tests to run. Should at least one of the follow: an ansible-playbooks directory containing one or more test directories each of which having a playbook.yml, a scripts directory containing one or more test directories each of which having a run-test.sh',
          name: 'TEST_DIR'
        ),
        string(
          defaultValue: '',
          description: 'Contains the CI_MESSAGE for a message bus triggered build.',
          name: 'CI_MESSAGE'
        )
      ]
    )
  ]
)

library(
  changelog: false,
  identifier: "multiarch-ci-libraries@${params.LIBRARIES_REF}",
  retriever: modernSCM([$class: 'GitSCMSource',remote: "${params.LIBRARIES_REPO}"])
)

List arches = params.ARCHES.tokenize(',')
def errorMessages = ''
def config = MAQEAPI.v1.getProvisioningConfig(this)


def make_host(def arch) {
  def target = MAQEAPI.v1.newTargetHost()
  def s390x = (arch == 's390x')
  def memory = s390x ? 4096 : 8192
  def cpus = s390x ? 2 : 4

  target.distro = "Fedora-28"
  target.arch = arch
  target.bkrHostRequires = [[tag:'memory', op:'>=',value: memory],
                            [tag:'processors',op:'>=',value: cpus],
                            [tag:'size',op:'>=',value: 50000]]
  return target
}

def target_hosts = arches.collect {make_host(it)}

MAQEAPI.v1.runTest(
  this,
  target_hosts,
  config,
  { host ->
    /*********************************************************/
    /* TEST BODY                                             */
    /* @param host               Provisioned host details.   */
    /*********************************************************/
    stage ('Download Test Files') {
      downloadTests()
    }

    stage ('Run Test') {
      runTests(config, host)
    }

    stage ('Archive Test Output') {
      archiveOutput()
    }

    /*****************************************************************/
    /* END TEST BODY                                                 */
    /* Do not edit beyond this point                                 */
    /*****************************************************************/
  },
  { Exception exception, def host ->
    def error = "Exception ${exception} occured on ${host.arch}\n"
    errorMessages += error
    if (host.arch.equals("x86_64") || host.arch.equals("ppc64le")) {
      currentBuild.result = 'FAILURE'
    }
  }
)
