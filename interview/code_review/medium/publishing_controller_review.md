# Publishing Controller

## Условие

Представьте, что вам на ревью пришел следующий код:

```java
@RestController
@RequestMapping("/public/v1/publishing")
@RequiredArgsConstructor
public class PublishingController {
    private final S3Service s3ServiceClient;
    private final ScanService scanService;
    private final MetadataRepository metadataRepository;

    @PostMapping(value = "/author/{publicationName}/version/{version}", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public void uploadApk(@PathVariable String author, @PathVariable String publicationName, @PathVariable Long version, @RequestPart("file") MultipartFile data) {
        String path = null;
        try {
            path = s3ServiceClient.uploadData(author, publicationName, version, data);
            scanService.validate(path);
        } catch (Exception e) {
            logger.error("error happened", e);
        }
         
        metadataRepository.save(author, publicationName, version, path);
    }
}
```

Опишите все проблемы и минусы, которые вы увидели в этом коде.

## Решение

### Форматирование

Первое, на что я бы обратил внимание, это неудачное форматирование кода, затрудняющее чтение: сигнатура метода контроллера, передача параметров.

Я бы предложил отформатировать в виде:

```java
@RestController
@RequestMapping("/public/v1/publishing")
@RequiredArgsConstructor
public class PublishingController {
    private final S3Service s3ServiceClient;
    private final ScanService scanService;
    private final MetadataRepository metadataRepository;

    @PostMapping(
        value = "/author/{publicationName}/version/{version}",
        consumes = MediaType.MULTIPART_FORM_DATA_VALUE
    )
    public void uploadApk(
        @PathVariable String author,
        @PathVariable String publicationName,
        @PathVariable Long version,
        @RequestPart("file") MultipartFile data
    ) {
        String path = null;
        try {
            path = s3ServiceClient.uploadData(
                author,
                publicationName,
                version,
                data
            );
            
            scanService.validate(path);
        } catch (Exception e) {
            logger.error("error happened", e);
        }
         
        metadataRepository.save(
            author,
            publicationName,
            version,
            path
        );
    }
}
```

### REST Endpoint

Сразу можно отметить, что передается `@PathVariable String author`, но сам endpoint не содержит такой параметр.

Также, в виде рекомендации, из-за соображений безопасности (так как у нас public API) я бы не передавал идентификатор автора в `URL`.

Следующее, что бросилось в глаза - это сам эндпоинт:

```javascript
/public/v1/publishing/author/{publicationName}/version/{version}
```

Он громоздкий и перегруженный, не понятен принцип его составления: для версий выделяется отдельный путь, а для публикаций нет (сразу после автора идет имя публикации).

Сам путь `/author` тоже выглядит странно и ничего не дает - ни идентификатора, ни каких-то связей.

```javascript
/public/v1/{authorId}/publications/{name}/{version}
```

В связи со всем вышесказанным, я бы посоветовал посмотреть в сторону:

```javascript
/public/v1/publications/{name}/{version}
```

Автора же мы будем получать из сессии или из jwt-токена, в зависимости от того как устроена авторизация пользователя.

Допускать возможность кому угодно пользоваться эндпоинтом я бы не рекомендовал.

Зависит от бизнес логики (из кода она не до конца ясна), но я бы рассмотрел подход, когда версия - это часть имени публикации.

Либо же передавать метаинформацию о файле в теле запроса вынести из `URL`:

```javascript
/public/v1/publications/{name}/
```

Добавим то, что ничего не возвращается при сохранении клиенту (void у метода). Возможно, стоит какой-то идентификатор добавить ресурса и так далее.

### Валидация

Отсутствует какая-либо валидация входных данных:

```java
        @PathVariable String author,
        @PathVariable String publicationName,
        @PathVariable Long version,
```

Строки могут быть пустыми, иметь разный регистр, версия может быть отрицательной, имя публикации тоже нужно валидировать по внутренним правилам и не допускать имен, не соответствующих правилами и паттернам.

### Логика в контроллере

Здесь сразу можно выделить один из главных минусов этого кода - это бизнес-логика в контроллере.

Почему это плохо?

Во-первых, это нарушение `S` из [SOLID](../../../patterns/SOLID.md), что приводит к нарушению взаимодействия слоев: сервисного и контроллера. Это усложняет понимание кода (когнитивную нагрузку), увеличивает шанс ошибки, усложняет тестирование кода (просто представьте как вы это будете тестировать) и так далее.

Отсюда совет - вынести бизнес-логику в отдельный сервис.

### Обработка исключений

Сама обработка исключений в виде:

```java
        } catch (Exception e) {
            logger.error("error happened", e);
        }
```

Является антипаттерном.

Лог не структурирован, сама проблема (ошибка) 'проглатывается' - даже если она происходит, то код продолжит работу и дойдет до сохранения в репозиторий и передачи вместо `path` значения `null`, а это уже нарушение бизнес-логики.

При этом, не учтено, что ошибка может произойти и на этапе сохранения в репозиторий - что уже будет обрабатываться по другим правилам, нежели ошибки из `s3ServiceClient` и `scanService`.

### Транзакционность

Отсутствие транзакционности в логике. Мы можем сохранить файл в S3, но упасть на валидации в `scanService` или при сохранении в репозиторий, что приведет к зомби-документу (в репозитории информации о нем не будет, но он будет в S3).

### Запутанная логика

Непонятная сигнатура `scanService#validate` - непонятно что делает, поэтому надо смотреть внутрь и тратить время.
Плюс непонятно, что произойдет, если валидация не пройдена.

Сама логика работы также вызывает вопросы - почему валидация идет после сохранения в S3, т.е. мы не доверяем предыдущему действию кода, а там наш клиент по сути.

По хорошему логику сохранения, валидации и реакции надо также выделить в отдельный сервис.

Также, я бы выделил в отдельную абстракцию идентификатор для сохранения, сейчас он непонятен:

```java
 path = s3ServiceClient.uploadData(
     author,
     publicationName,
     version,
     data
 );
```

Что является уникальным идентификатором? Какие параметры обеспечивают уникальность контента?

Я бы посоветовал вынести это в отдельный класс:

```java
public class ContentId {
    // ...
}
```

И уже там валидацию на данные добавил бы, тем самым, сузив зону ответственности в сервисе.

Соответственно, загрузка бы выглядела так:

```java
 path = s3ServiceClient.uploadData(
     contentId,
     data
 );
```
