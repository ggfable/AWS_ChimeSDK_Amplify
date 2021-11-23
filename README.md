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
       
Заменить */amplify/backend/function/reactSampleLambda/src/index.js с помощью кода, представленного в файле [index.js](https://github.com/sFablee/AWS_ChimeSDK_Amplify/blob/main/index.js). 
Затем вернитесь к терминалу и нажмите Enter, чтобы завершить настройку.

    2. Добавление категории 'api' создаст GraphQL API в AWS AppSync. Добавьте категорию 'api' с помощью Amplify CLI и ответьте на следующие вопросы:
        
        $ amplify add api

        ? Please select from one of the below mentioned services: GraphQL
        ? Provide API name: reactSampleApi
        ? Choose the default authorization type for the API: API key
        ? Enter a description for the API key:
        ? After how many days from now the API key should expire (1-365): 7
        ? Do you want to configure advanced settings for the GraphQL API No, I am done.
        ? Do you have an annotated GraphQL schema? No
        ? Choose a schema template: Single object with fields (e.g., “Todo” with ID, name, description)

        ? Do you want to edit the schema now? Yes
        
### Когда редактор откроется, замените файл схемы содержимым следующей схемы:
        
        type Meeting @model(mutations: {create: "createMeetingGraphQL", delete: "deleteMeetingGraphQL"}, subscriptions: null) @key(fields: ["title"]){
          meetingId: String!
          title: String!
          data: String!
        }

        type Attendee @model(mutations: {create: "createAttendeeGraphQL", delete: "deleteAttendeeGraphQL"}, subscriptions: null) @key(fields: ["attendeeId"]){
          attendeeId: String!
          name: String!
        }

        type Query {
          createChimeMeeting(title: String, name: String, region: String): Response @function(name: "reactSampleLambda-${env}")
          joinChimeMeeting(meetingId: String, name: String): Response @function(name: "reactSampleLambda-${env}")
          endChimeMeeting(meetingId: String): Response  @function(name: "reactSampleLambda-${env}")
        }

        type Response {
          statusCode: String!
          headers: String
          body: String
          isBase64Encoded: String
        }
        
    3. Затем, чтобы перенести изменения в облако, выполните следующую команду - вы можете принять все значения по умолчанию в prompt:
    
        $ amplify push
        
    4. Наконец, измените политику IAM Role, включив в нее Amazon Chime Full Access, чтобы позволить вашей функции Lambda вызывать API Amazon Chime:
        
        - Перейдите в AWS Console, используя ту же учетную запись, которую вы использовали для настройки Amplify.
        - Перейдите в раздел IAM
        - Нажмите на Роли в боковом меню.
        - Найдите созданную роль и щелкните по ее названию - {projectName}LambdaRoleXXX-{environment}
        - Нажмите на кнопку "Attach Policies".
        - Введите в поле поиска: AmazonChimeFullAccess
        - Нажмите на флажок для: AmazonChimeFullAccess
        - Нажмите на "Attach Policy" в правом нижнем углу экрана
 
### Подключение фронт-энда к бэк-энду с помощью GraphQL

    После того, как вы перешли на учетную запись AWS, вы можете проверить сгенерированные API GraphQL внутри каталога ./src/graphql/*. 
    Вы должны увидеть ./src/graphql/mutations.js и ./src/graphql/queries.js. Эти файлы содержат API GraphQL, которые были сгенерированы программой 
    Amplify, которые вы теперь можете использовать в своем приложении
    
    1. Создайте новый каталог ./src/utils/
    2. Создайте новый файл api.ts в разделе ./src/utils/api.ts. Скопируйте приведенный ниже код в новый файл:
    
        import { API, graphqlOperation } from 'aws-amplify';
        import { createAttendeeGraphQL, createMeetingGraphQL, deleteMeetingGraphQL } from '../graphql/mutations';
        import { createChimeMeeting, getAttendee, endChimeMeeting, getMeeting, joinChimeMeeting } from '../graphql/queries';


        export async function createMeeting(title: string, attendeeName: string, region: string) {
          const joinInfo: any = await API.graphql(graphqlOperation(createChimeMeeting, {title: title, name: attendeeName, region: region }));
          const joinInfoJson = joinInfo.data.createChimeMeeting;
          const joinInfoJsonParse = JSON.parse(joinInfoJson.body);
          return joinInfoJsonParse;
        }

        export async function joinMeeting(meetingId: string, name: string) {
          const joinInfo: any = await API.graphql(graphqlOperation(joinChimeMeeting, {meetingId: meetingId, name: name}));
          const joinInfoJson = joinInfo.data.joinChimeMeeting;
          const joinInfoJsonParse = JSON.parse(joinInfoJson.body);
          return joinInfoJsonParse;
        }

        export async function endMeeting(meetingId: string) {
          const endInfo: any = await API.graphql(graphqlOperation(endChimeMeeting, {meetingId: meetingId}));
          const endInfoJson = endInfo.data.endChimeMeeting;
          await API.graphql(graphqlOperation(deleteMeetingGraphQL, {input: {title: meetingId}}));
          return endInfoJson;
        }

        export async function addMeetingToDB(title: string, meetingId: string, meetingData: string) {
          await API.graphql(graphqlOperation(createMeetingGraphQL, {input: {title: title, meetingId: meetingId, data: meetingData, }}));
        }

        export async function addAttendeeToDB(attendeeID: string, attendeeName: string) {
          await API.graphql(graphqlOperation(createAttendeeGraphQL, {input: {attendeeId: attendeeID, name: attendeeName }}));
        }

        export async function getMeetingFromDB(title: string) {
          const meetingInfo = await API.graphql(graphqlOperation(getMeeting, {title: title }));
          return meetingInfo;
        }

        export async function getAttendeeFromDB(attendeeId: string) {
          const attendeeInfo = await API.graphql(graphqlOperation(getAttendee, {attendeeId: attendeeId }));
          return attendeeInfo;
        }
        
    3. Откройте файл ./src/components/MeetingForm.tsx. Скопируйте эту реализацию функции getAttendeeCallback и joinMeeting в поле 
       joinMeeting в ./src/components/MeetingForm.tsx.
       
        const clickedJoinMeeting = async (event: FormEvent) => {
          event.preventDefault();

          meetingManager.getAttendee = getAttendeeCallback();
          const title = meetingTitle.trim().toLocaleLowerCase();
          const name = attendeeName.trim();

        // Fetch the Meeting via AWS AppSync - if it exists, then the meeting has already
        // been created, and you just need to join it - you don't need to create a new meeting
          const meetingResponse: any = await getMeetingFromDB(title);
          const meetingJson = meetingResponse.data.getMeeting;
          try {
            if (meetingJson) {
              const meetingData = JSON.parse(meetingJson.data);
              const joinInfo = await joinMeeting(meetingData.MeetingId, name);
              await addAttendeeToDB(joinInfo.Attendee.AttendeeId, name);

              await meetingManager.join({
                meetingInfo: meetingData,
                attendeeInfo: joinInfo.Attendee
              });
            } else {
              const joinInfo = await createMeeting(title, name, 'us-east-1');
              await addMeetingToDB(title, joinInfo.Meeting.MeetingId, JSON.stringify(joinInfo.Meeting));       await addAttendeeToDB(joinInfo.Attendee.AttendeeId, name);

              await meetingManager.join({
                meetingInfo: joinInfo.Meeting,
                attendeeInfo: joinInfo.Attendee
              });
            }
          } catch (error) {
            console.log(error);
          }

          // At this point you can let users setup their devices, or start the session immediately
          await meetingManager.start();
        };
        
    4. Теперь вы можете запустить клиент собраний локально. Для этого перейдите в каталог корневого уровня репозитория и выполните следующую команду:
    
        $ npm install && npm run build && npm run start
        
## Очистка

    Чтобы избежать непредвиденных расходов в результате развертывания демонстрационного приложения, важно удалить все ресурсы в вашей учетной записи AWS, 
    которые вы не используете.

    Когда вы впервые вызвали 'amplify init', вы создали локальную среду Amplify. Вы можете иметь несколько окружений на вашем аккаунте AWS, 
    но вызов 'amplify remove <category>' удаляет соответствующие ресурсы в вашем локальном окружении Amplify.
    
        $ amplify remove { api | function }
        
    Удаление категории в локальной среде Amplify не влияет на ресурсы, созданные в облаке. После удаления категории локально, удалите ресурсы из учетной записи AWS, 
    опубликовав эти изменения в облаке:
    
        $ amplify push
        
    Кроме того, вы можете очистить всю среду Amplify в облаке и все локальные файлы, созданные Amplify, выполнив команду:
    
        $ amplify delete
