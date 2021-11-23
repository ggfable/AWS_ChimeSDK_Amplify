### 1:
    $ npx create-react-app react-sample --template typescript
    
### 2:
    $ cd react-sample
    
### 3:
    $ npm install
    
### 3:
    $ npm install --save amazon-chime-sdk-component-library-react amazon-chime-sdk-js styled-components styled-system aws-amplify && npm install --save-dev @types/styled-components

## Установка и настройка Amplify

### 1:
    $ npm install -g @aws-amplify/cli
    
### 2:
    $ amplify configure
    
### 3:
    $ amplify init
    
        ? Enter a name for the project: (amplifyDemo) 
        ? Enter a name for the environment: dev 
        ? Choose your default editor: Visual Studio Code
        ? Choose the type of app that youre building: javascript 
        ? What JavaScript framework are you using: React
        ? Source Directory Path: src 
        ? Distribution Directory Path: dist 
        ? Build Command: npm run-script build 
        ? Start Command: npm run-script serve 
        ? Do you want to use an AWS profile? Yes 
        ? Please choose the profile you want to use [Your AWS Profile] - Choose the AWS Profile configured in the previous step
        
## Создайте frontend часть приложения, используя библиотеку React-компонентов Amazon Chime SDK
    1. Удалите все лишние файлы в ./public за исключением index.html
    2. Заменить ./public/index.html со следующим начальным кодом html:
       
        <!DOCTYPE html>
        <html>
          <head>
            <meta charset="UTF-8" />
            <meta name="viewport" content="width=device-width, initial-scale=1.0" />
            <meta http-equiv="X-UA-Compatible" content="ie=edge" />
            <title>Amazon Chime SDK Amplify Demo</title>
            <link rel="icon" href="data:,">
          </head>

          <body>
            <div id="root"></div>
          </body>
        </html>
        
    3. Создайте новый каталог ./src/components/
    4. Создайте компонент Meeting в src/components/Meeting.tsx. Этот начальный код показывает, как использовать ControlBar, AudioInputControl, VideoInputControl,
       и AudioOutputControl components для Amazon Chime SDK React Library. Скопируйте следующий код в src/components/Meeting.tsx:
        
        import React, { FC } from 'react';

        import {
          AudioInputControl,
          AudioOutputControl,
          ControlBar,
          ControlBarButton,
          Phone,
          useMeetingManager,
          MeetingStatus,
          useMeetingStatus,
          VideoInputControl,
          VideoTileGrid
        } from 'amazon-chime-sdk-component-library-react';
        import { endMeeting } from '../utils/api';

        const Meeting: FC = () => {
          const meetingManager = useMeetingManager();
          const meetingStatus = useMeetingStatus();

          const clickedEndMeeting = async () => {
            const meetingId = meetingManager.meetingId;
            if (meetingId) {
              await endMeeting(meetingId);
              await meetingManager.leave();
            }
          }

          return (
              <div style={{marginTop: '2rem', height: '40rem', display: 'flex', flexDirection: 'column', alignItems: 'center', justifyContent: 'center'}}>
                <VideoTileGrid/>
                {meetingStatus === MeetingStatus.Succeeded ? 
                  <ControlBar
                    layout="undocked-horizontal"
                    showLabels
                  >
                    <AudioInputControl />
                    <VideoInputControl />
                    <AudioOutputControl />
                    <ControlBarButton icon={<Phone />} onClick={clickedEndMeeting} label="End" />
                  </ControlBar> 
                  :
                  <div/>
                }
              </div>
          );
        };

        export default Meeting;
        
    4. Создайте компонент MeetingForm в ./src/components/MeetingForm.tsx. Этот компонент отвечает за получение названия собрания, имени участника и региона. 
       Отправка формы вызывает API для создания или присоединения к существующей встрече с использованием данных из формы. Tдержатель функции joinMeeting будет 
       будет заменено на следующем шаге. Скопируйте следующий код в ./src/components/MeetingForm.tsx:
       
       import React, { ChangeEvent, FC, FormEvent, useState } from 'react';

        import {
          Flex,
          FormField,
          Input,
          PrimaryButton,
          useMeetingManager,
        } from 'amazon-chime-sdk-component-library-react';
        import { addAttendeeToDB, addMeetingToDB, createMeeting, getAttendeeFromDB, getMeetingFromDB, joinMeeting } from '../utils/api';

        const MeetingForm: FC = () => {
          const meetingManager = useMeetingManager();
          const [meetingTitle, setMeetingTitle] = useState('');
          const [attendeeName, setName] = useState('');

          function getAttendeeCallback() {
            return async (chimeAttendeeId: string, externalUserId?: string) => {
              const attendeeInfo: any = await getAttendeeFromDB(chimeAttendeeId);
              const attendeeData = attendeeInfo.data.getAttendee;
              return {
                name: attendeeData.name
              };
            }
          }

        //Placeholder - we'll replace this function implementation later
          const clickedJoinMeeting = async (event: FormEvent) => {
            event.preventDefault();
          };

          return (
            <form>
              <FormField
                field={Input}     
                label='Meeting Id'
                value={meetingTitle}
                fieldProps={{
                  name: 'Meeting Id',
                  placeholder: 'Enter a Meeting ID',
                }}
                onChange={(e: ChangeEvent<HTMLInputElement>) => {
                  setMeetingTitle(e.target.value);
                }}
              />
              <FormField
                field={Input}
                label="Name"
                value={attendeeName}
                fieldProps={{
                  name: 'Name',
                  placeholder: 'Enter your Attendee Name'
                }}
                onChange={(e: ChangeEvent<HTMLInputElement>) => {
                  setName(e.target.value);
                }}
              />
              <Flex
                container
                layout="fill-space-centered"
                style={{ marginTop: '2.5rem' }}
              >
                  <PrimaryButton label="Join Meeting" onClick={clickedJoinMeeting} />
              </Flex>
            </form>
          );
        };

        export default MeetingForm;
       
    5. Удалите весь посторонний код в ./src за исключением aws-exports.js и ./index.tsx.
    6. Настройте Amplify в клиентском коде так, чтобы мы могли GraphQL APIs. Заменить ./index.tsx со следующим начальным кодом:
    
        import ReactDOM from 'react-dom';
        import { ThemeProvider } from 'styled-components';
        import {
          MeetingProvider,
          lightTheme
        } from 'amazon-chime-sdk-component-library-react';
        import Meeting from './components/Meeting';
        import MeetingForm from './components/MeetingForm';

        import Amplify from 'aws-amplify';
        import awsconfig from './aws-exports';
        Amplify.configure(awsconfig);

        window.addEventListener('load', () => {
          ReactDOM.render(
          <ThemeProvider theme={lightTheme}>
            <MeetingProvider>
              <MeetingForm />
              <Meeting/>
            </MeetingProvider>
          </ThemeProvider>
          , document.getElementById('root'));
        });
  
## Создание ресурсов back end для вашего React приложения с помощью Amplify

       Amplify предоставляет команду CLI для создания предопределенного набора ресурсов задней части, называемого "category". Добавив категории 'function' и 'api' к вашему
       Приложение Amplify, Amplify автоматически генерирует необходимые внутренние ресурсы, необходимые для работы вашего приложения без сервера. 
       Когда вы добавляете категории и делаете обновления конфигурации backend с помощью Amplify CLI, конфигурация в ./aws-exports.js будет обновляться автоматически.
    
    1. Добавление категории 'function' настраивает функцию Lambda, которая может обрабатывать запросы API. С корневого уровня проекта добавьте категорию 'function' 
       с помощью  Amplify CLI и ответьте на предложенные вопросы следующим образом:
       
        $ amplify add function

        ? Select which capability you want to add: Lambda function (serverless function)
        ? Provide an AWS Lambda function name: reactSampleLambda
        ? Choose the runtime that you want to use: NodeJS
        ? Choose the function template that you want to use: Hello World
        ? Do you want to configure advanced settings? No
        ? Do you want to edit the local lambda function now? Yes
       
    2. Заменить */amplify/backend/function/reactSampleLambda/src/index.js с помощью кода, представленного в файле [index.js](https://github.com/sFablee/AWS_ChimeSDK_Amplify/blob/main/index.js). 
       Затем вернитесь к терминалу и нажмите Enter, чтобы завершить настройку.
