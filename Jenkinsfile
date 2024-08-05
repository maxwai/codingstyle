node('java-agent') {
    stage ('Checkout') {
        checkout scm
    }

    stage ('Git mining') {
        discoverGitReferenceBuild()
        mineRepository()
        gitDiffStat()
    }

    stage ('Build, Test, and Static Analysis') {
        withMaven(mavenLocalRepo: '/var/data/m2repository', mavenOpts: '-Xmx768m -Xms512m') {
            sh 'mvn -V -e clean verify -Dgpg.skip -Pci'
        }

        recordCoverage(tools: [[parser: 'JACOCO'], [parser: 'METRICS', pattern: "target/pmd-metrics-java/pmd.xml"]],
                id: 'jacoco', name: 'JaCoCo Coverage',
                sourceCodeRetention: 'EVERY_BUILD',
                qualityGates: [
                    [threshold: 60.0, metric: 'LINE', baseline: 'PROJECT', unstable: true],
                    [threshold: 60.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true]])

        recordCoverage(tools: [[parser: 'METRICS', pattern: "target/pmd-metrics-java/pmd.xml"]],
                id: 'metrics', name: 'Metrics',
                sourceCodeRetention: 'EVERY_BUILD')
    }

    stage ('Collect Maven Warnings') {
        recordIssues tool: mavenConsole()
    }
}
