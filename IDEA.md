My company CI pipeline has these steps: checkout scm, run unit test, scan code quality, check quality gate, execute checker, build & push liquibase image, build image, scan code security, check security gate, scan image security, scan artifact security, check sca gate.

My company is a bank, which is very serious about security. That CI pipeline is used for spring boot microservices running on kubernetes, deployed automatically using ArgoCD.

I want to build a good similar pipeline, but for Django - Python, focus more on functional, speed, can be used for a startup company.
