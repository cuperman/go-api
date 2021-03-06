'use strict';

const path = require('path');
const fs = require('fs');
const aws = require('./aws.json');

const AWS_PROFILE = aws.profile;
const AWS_S3_BUCKET = aws.codeBucket;

const PACKAGE_DIR  = 'package';
const PACKAGE_NAME = 'go-api';

const TEMPLATE_FILE = 'template.json';
const DEPLOYMENT_FILE = 'deployed';

const STACK_NAME   = 'go-api';
const STACK_FILE = 'stacks.json';

const SOURCE_FILES = [
  'app',
  'package*.json'
];

namespace('code', () => {
  const PACKAGE_PATH = path.resolve(PACKAGE_DIR, PACKAGE_NAME);
  const CODE_FILE = `${PACKAGE_NAME}.zip`;
  const CODE_PATH = path.resolve(PACKAGE_DIR, CODE_FILE);

  directory(PACKAGE_PATH);

  task('copy files', [PACKAGE_PATH], () => {
    const sourceFileList = new jake.FileList();
    sourceFileList.include(SOURCE_FILES);
    sourceFileList.toArray().forEach(file => {
      jake.cpR(file, PACKAGE_PATH);
    });
  });

  task('npm install', ['copy files'], { async: true }, () => {
    const cmd = `npm install --prefix ${PACKAGE_PATH} --production`;
    jake.logger.log(cmd);
    jake.exec(cmd, { printStdout: true, printStderr: true }, () => {
      complete();
    });
  });

  file(CODE_PATH, ['npm install'], { async: true }, () => {
    const cmd = `cd ${PACKAGE_PATH}; zip -r ../${CODE_FILE} *`;
    jake.logger.log(cmd);
    jake.exec(cmd, { printStdout: true, printStderr: true }, () => {
      complete();
    });
  });

  task('build', [CODE_PATH]);

  task('clean', [], () => {
    jake.rmRf(PACKAGE_DIR);
  });
});

namespace('stack', () => {
  const CODE_FILE = `${PACKAGE_NAME}.zip`;
  const CODE_PATH = path.resolve(PACKAGE_DIR, CODE_FILE);
  const OUTPUT_TEMPLATE_PATH = path.resolve(PACKAGE_DIR, TEMPLATE_FILE);
  const DEPLOYMENT_PATH = path.resolve(PACKAGE_DIR, DEPLOYMENT_FILE);
  const STACK_PATH = path.resolve(PACKAGE_DIR, STACK_FILE);

  file(OUTPUT_TEMPLATE_PATH, [CODE_PATH, TEMPLATE_FILE], { async: true }, () => {
    const cmd = `aws --profile ${AWS_PROFILE} cloudformation package --template-file ${TEMPLATE_FILE} --s3-bucket ${AWS_S3_BUCKET} --output-template-file ${OUTPUT_TEMPLATE_PATH} --use-json`;
    jake.logger.log(cmd);
    jake.exec(cmd, { printStdout: true, printStderr: true }, () => {
      complete();
    });
  });

  task('package', [OUTPUT_TEMPLATE_PATH]);

  file(DEPLOYMENT_PATH, [OUTPUT_TEMPLATE_PATH], { async: true }, () => {
    const cmd = `aws --profile ${AWS_PROFILE} cloudformation deploy --template-file ${OUTPUT_TEMPLATE_PATH} --stack-name ${STACK_NAME} --capabilities CAPABILITY_IAM && touch ${DEPLOYMENT_PATH}`;
    jake.logger.log(cmd);
    jake.exec(cmd, { printStdout: true, printStderr: true }, () => {
      complete();
    });
  });

  file(STACK_PATH, [DEPLOYMENT_PATH], { async: true }, () => {
    const cmd = `aws --profile ${AWS_PROFILE} cloudformation describe-stacks --stack-name ${STACK_NAME} | tee ${STACK_PATH}`;
    jake.logger.log(cmd);
    jake.exec(cmd, { printStdout: true, printStderr: true }, () => {
      complete();
    });
  });

  task('deploy', [STACK_PATH]);

  task('events', [], { async: true }, () => {
    const cmd = `aws --profile ${AWS_PROFILE} cloudformation describe-stack-events --stack-name ${STACK_NAME}`;
    jake.logger.log(cmd);
    jake.exec(cmd, { printStdout: true, printStderr: true }, () => {
      complete();
    });
  });

  task('delete', [], { async: true }, () => {
    const cmd = `aws --profile ${AWS_PROFILE} cloudformation delete-stack --stack-name ${STACK_NAME}`;
    jake.logger.log(cmd);
    jake.exec(cmd, { printStdout: true, printStderr: true }, () => {
      complete();
    });
  });
});

desc('Build the code');
task('build', ['code:build']);

desc('Clean up (remove the build files)');
task('clean', ['code:clean']);

desc('Deploy the stack on AWS');
task('deploy', ['stack:deploy']);

desc('Destroy the stack on AWS');
task('destroy', ['stack:delete']);
