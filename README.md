# firebase-deploy

[CircleCI](https://circleci.com) [Orb](https://circleci.com/docs/2.0/orb-intro/#section=configuration) for deploying to [Firebase](https://firebase.google.com/)

## Usage

[Read the docs](https://circleci.com/orbs/registry/orb/azdevs/firebase-deploy)

Example `.circleci/config.yaml`

```yaml
version: 2.1
orbs:
  firebase-deploy: azdevs/firebase-deploy@1.0.0
jobs:
  build:
    docker:
      - image: circleci/node:lts
    steps:
      - checkout
      - restore_cache:
          key: v1-yarn-{{ checksum "yarn.lock" }}
      - run: yarn install
      - save_cache:
          key: v1-yarn-{{ checksum "yarn.lock" }}
          paths:
            - ./node_modules
      - run: yarn build
      - persist_to_workspace:
          root: /tmp/workspace
          paths:
            - dist
            - firebase.json
            - .firebaserc
            - firestore.rules
            - firestore.indexes.json
  deploy-staging:
    docker:
      - image: circleci/node:lts
    steps:
      - attach_workspace:
          at: /tmp/workspace
      - firebase-deploy/deploy:
          token: $FIREBASE_DEPLOY_TOKEN
          alias: staging # name of an alias from your .firebaserc
workflows:
  version: 2
  build-and-deploy-staging:
    jobs:
      - build
      - deploy-staging:
          requires:
            - build
          filters:
            branches:
              ignore: master
```
