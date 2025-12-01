_Общие назначение проекта можно найти в папке 'definition'_

# ML System Design Doc

## Дизайн ML систем - Productive Social MVP

*Шаблон ML System Design Doc от телеграм-канала [Reliable ML](https://t.me/reliable_ml)*

## 1. Зачем идем в разработку продукта?
 - бизнес-цель Productive Social is to become the best application for productivity.
we aim not only no help people increase their productive, but to develop a system capable of
studying individual productivity, develop an understanding on individual habits (productive habits or not)
and based on these provide informational reports to the user along with recommendations on how to be more productive.
 - Productive Social is better than current available application in three ways:
   - It will focus on all aspects of self-productivity;
   - It will provide a ML system that learns on specific individual data (data-collection through the many features
 that Productive Social will include)
   - It will have a social media aspect, designed to drive up user-engage and retention.

## 2. Бизнес-требования и ограничения
 - Develop an all-in-one application for anything productivity, the application should 
provide different functionality as stand-alone applications, as well as completely integrate
all features togethers. Apart from the social media aspect, all features should not require an internet 
connection to work.
 - The MVP should provide: 
   - A task management feature;
   - A note-taking feature;
   - A pomodoro feature;
   - A journal feature
each of these features should provide all the basic capabilities of an equivalent application. 
it would be very time expensive to go in details on the requirements for each feature.
   - Lastly a social media aspect, should include, but not limited to:
     - просмотр отчётов о продуктивности других пользователей и их прогресса
     - поддержка обмена медиа (без кастомных настроек)
     - личный профиль пользователя
 - Ограничения
   - Использование платных технологических платформ (например, AWS).
   - Соблюдение требований GDPR и других нормативных актов о защите данных. 
   - Время реализации — не менее 6 месяцев.
 - Описание бизнес-процесса пилота, насколько это возможно - как именно мы будем использовать 
модель в существующем бизнес-процессе?
   - Productive Social будет доступно через Google Play Store для андройда и App Store для iOS.
 - Что считаем успешным пилотом? Критерии успеха и возможные пути развития проектa?
   - Трудно сказать что является успех для данного проекта, пока я уверен что смогу разработать приложение
 не могу сказать если систем ML будет работать как желается.
   Если немного поговорим o фиче 'Journaling' the ML systems should be able to provide a daily journal
 with reports formed based on the usage of the other features Productive Social provides, thus should
 be able to tell exactly what the user worked-on on a given day, how many hours in spent on it and how long
 it took him to achieve a certain goal. but if a user doesn't use the application for two or more days
 then the journal feature should still (predict) and provide an optional journal entry for those two days. If
 there's information on the task manager, note-taking or previous journal entries about what the user probably did
 for the past two days then the ML system should be aware of it, and provide an automatic journal entry.
 Если приложение будет работать как ожиданно - это успех.
 
## 3. Что входит в скоуп проекта, что не входит
 - Входит:
   - A task management feature;
   - A note-taking feature;
   - A pomodoro feature;
   - A journal feature
   - A Social Media feature.
 - Не входит:
   - a full-pledged social media feature. the social media features of this project should be 
   limited to appreciating the productivity of other people
   (even seeing how long it took them to complete a certain task), but, not to:
     - directly message or privately interact with other users
     - any other feed besides the main feed
 - На закрытие каких БТ подписываемся в данной итерации
   - На MVP, на мобильное приложение ожидается реализаций все выше перечислены фичи, 
   для Data Scientist ожидается реализации фичи дневника 
 - Описание результата с точки зрения качества кода и воспроизводимости решения Data Scientist
   - Результат должен соответствовать:
     - Высокому качеству кода (читаемость, чистота, документация)
     - Воспроизводимости (использование версионирования, контейнеризация, автоматизация)
 - Описание планируемого технического долга (что оставляем для дальнейшей продуктивизации) Data Scientist
   - В будущем планируется внедрение ИИ во всех фичи приложения.

## 4. Предпосылки решения
 - используемые блоки данных:
   - внутренние базы данных, habit and behavior datasets from Kaggle;
   - User engagement and usage patterns - Public datasets on mobile app engagement or synthetic datasets created for app analytics.
   - Motivational and psychological data - Research datasets or surveys related to psychology and motivation.
   - External data for context - Public weather datasets, calendar APIs.
   - Through usage Productive Social will take in as much data as possible (внутри закона).
 - Гранулярность модели:
   - Хотя модели будут для общего назначения, при использовании модели должни обучаться на данных определенного юзера

## Методология
### TODO()