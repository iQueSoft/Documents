*Знакомство с расширенными возможностями iOS по обработке видео на примере
стандартного семпла AVCustomEdit. А также, пример расширения этого семпла.*

 

В этой статье хочу представить очень интересный семпл AVCustomEdit, который
предоставляет нам Apple (ссылка
<https://developer.apple.com/library/ios/samplecode/AVCustomEdit/Introduction/Intro.html#//apple_ref/doc/uid/DTS40013411-Intro-DontLinkElementID_2>
). Он демонстрирует удивительную фичу – эффекты пререходов между
видиофрагментами. И, я хочу расмотреть принцип его работы, а также поделится
собственной доработкой этого сепла, котрая упрощает и расширяет функционал
исходника.

Что же происходит в этой программе? Какие алгоритмы и технологии используются?
Как всегда, кода очень много, много классов, связей между классами и все кажется
сложным и запутанным.

Начнем с описания основной цели семпла. Он объединяет два видеофайла в один и во
время перехода видео не просто перескакивает, а плавно перетекает с одного
фрагмента в другой с добавлением некоторого эффекта. Перед тем как начать
рассмотрение сложной части, а именно добавления эффектов, хочу вспомнить более
простую часть – это обычное объединение двух видеофайлов в один. На основе
простого организовывается более сложное. Возвращаясь к первой статье: «В самом
распространенном виде, видеофайл состоит из двух треков. Это видеотрек и
аудиотрек. Если у нас есть несколько видеофайлов, то мы естественно имеем
несколько видеотреков и несколько аудиотреков. А нам нужно получить один общий
видик, то есть один общий видеотрек и один аудиотрек. Для этого нам нужно все
исходные видео файлы разделить на треки, а эти треки соединить последовательно
друг за другом. Получившиеся в результате видео и аудио треки нужно объединить в
видеофайл.

Как это делается в AVFoundation? Какие классы используются?

Для этого используются классы AVAsset, для представления файла, и AVAssetTrack,
для представления трека. Самим процессом воссоздания видеофайла с треков
занимается класс AVAssetExportSession. При этом видеотреки обьединяются
отдельно, а аудиотреки отдельно в AVMutableCompositionTrack. Две композиции
треков обьединяются в общую композицию AVMutableComposition, которую использует
сессия. Указываем экспорт сессии путь в файловой системе для нового файла и
запускаем ее на обработку.»

В семпле AVCustomEdit все устроенно именно так, но с относительно небольшим
дополнением. Этим дополнением является протокол AVVideoCompositing. Здесь видео
объединяются не просто один за другим, а с небольшим наложением друг на друга.
То есть, в месте перехода с одного видео на другое, последние несколько кадров
первого видео накладываются на несколько начальных кадров второго. В свою
очередь, AVVideoCompositing дает возможность нам в коде получать эти два кадра,
обьединять, добавлять эффекты и возвращать в экспорт сессию, что б она добавила
их в новый видеофайл.

Как это выглядит в коде? Вернемся к AVAssetExportSession, у этого класса есть
два интересных поля audioMix и videoComposition. Если кратно о них, то они нужны
для дополнительных настроек обработки видео и аудиотреков. Думаю понятно, что
audioMix необходим для звука, а videoComposition – для видео. Но нам audioMix
пока не нужен. Остановимся на videoComposition, тип этой проперти
AVVidoeComposition или AVMutableVideoComposition. И так, для дополнительных
возможностей эксперт сессии в семпле создается экземпляр
AVMutableVideoComposition и добавляется в сессию. Начиная с iOS 7.0 у этого
класса была добавлена проперти customVideoCompositorClass. Тип не важен, важно
что этот композитор должен поддерживать протокол AVVideoCompositing, о котором я
уже упоминал.

![](<report_3_image_1.png>)

Объект класса, который поддерживает протокол AVVideoCompositiong отвечает за
получение двух буферов, их обработку, то есть добавления эффекта и возвращение в
сессию нового буфера. В семпле это класс APLCustomVideoCompositor. Для создания
различных эффектов переходов, необходимо создавать по отдельному такому классу
для каждого эффекта. Оптимальный подход, это создать базовый класс, а новые
эффекты делать классами наследниками. В исходном семпле таких наследников два:
APLCrossDissolveCompositor и ALPDiagonalWipeCompositor.

Сама обработка буферов выполняется средствами OpenGL, по причине, которая была
детальней рассмотрена в предыдущей статье, то бишь по причине большей
производительности. Мы получаем два буфера, их конвертируем в текстуры, которые
вместе с шейдером, программой попиксельной обработки в OpenGL, отправляем на
обработку в GPU. После обработки в OpenGL мы получаем новую текстуру, которую
возвращаем обратно в экспорт сессию. В семпле за это отвечает класс
APLOpenGLRenderer. Для создания конкретных эффектов были добавлены два класса
наследника APLDiagonalWipeRenderer и APLCrossDissolveRenderer. APLOpenGLRenderer
является полем класса APLCustomVideoCompositor. APLCrossDissolveCompositor и
ALPDiagonalWipeCompositor различаются тем, что инициализирую каждый свой
рендерер.

Вернемся к протоколу AVVideoCompositiong, в APLCustomVideoCompositor реализованы
методы этого протокола:

Настройка типов входных и выходных буферов.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- (NSDictionary *)sourcePixelBufferAttributes
{
    return @{ (NSString *)kCVPixelBufferPixelFormatTypeKey : [NSNumber numberWithUnsignedInt:kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange],
(NSString*)kCVPixelBufferOpenGLESCompatibilityKey : [NSNumber numberWithBool:YES]};
}

- (NSDictionary *)requiredPixelBufferAttributesForRenderContext
{
    return @{ (NSString *)kCVPixelBufferPixelFormatTypeKey : [NSNumber numberWithUnsignedInt:kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange],
(NSString*)kCVPixelBufferOpenGLESCompatibilityKey : [NSNumber numberWithBool:YES]};
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Выбор контекста в котором будут кэшироваться текстуры во время рендеринга.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- (void)renderContextChanged:(AVVideoCompositionRenderContext *)newRenderContext
{
    dispatch_sync(_renderContextQueue, ^() {
        _renderContext = newRenderContext;
        _renderContextDidChange = YES;
    });
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Метод в котором мы получаем буферы и возвращаем новый.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- (void)startVideoCompositionRequest:(AVAsynchronousVideoCompositionRequest *)request
{
    @autoreleasepool {
        dispatch_async(_renderingQueue,^() {
            // Check if all pending requests have been cancelled
            if (_shouldCancelAllRequests) {
                [request finishCancelledRequest];
            } else {
                NSError *err = nil;
                // Get the next rendererd pixel buffer
                CVPixelBufferRef resultPixels = [self newRenderedPixelBufferForRequest:request error:&err];
                if (resultPixels) {
                    // The resulting pixelbuffer from OpenGL renderer is passed along to the request
                    [request finishWithComposedVideoFrame:resultPixels];
                    CFRelease(resultPixels);
                } else {
                    [request finishWithError:err];
                }
            }
        });
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Если точнее, то мы получаем реквест, с которого мы можем вытянуть буферы.
Обратите внимание на метод newRenderedPixelBufferForRequest:error: , в него
передается сам реквест, а он уже возвращает новый буфер.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
-(CVPixelBufferRef)newRenderedPixelBufferForRequest:(AVAsynchronousVideoCompositionRequest *)request error:(NSError **)errOut
{
    CVPixelBufferRef dstPixels = nil;

    ...

    // Source pixel buffers are used as inputs while rendering the transition
    CVPixelBufferRef foregroundSourceBuffer = [request sourceFrameByTrackID:currentInstruction.foregroundTrackID];
    CVPixelBufferRef backgroundSourceBuffer = [request sourceFrameByTrackID:currentInstruction.backgroundTrackID];
    // Destination pixel buffer into which we render the output
    dstPixels = [_renderContext newPixelBuffer];

    ...

    [_oglRenderer renderPixelBuffer:dstPixels usingForegroundSourceBuffer:foregroundSourceBuffer andBackgroundSourceBuffer:backgroundSourceBuffer forTweenFactor:tweenFactor];

    return dstPixels;
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Рендеринг выполняется объектом класса APLOpenGLRenderer \_oplRenderer, в который
передаются два полученных буфера и зарезервированное место в текущем контексте
для нового буфера, будет возвращен. Если посмотреть реализацию метода, который
здесь вызывается у рендерера, в базовом классе, то увидим следующее:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- (void)renderPixelBuffer:(CVPixelBufferRef)destinationPixelBuffer usingForegroundSourceBuffer:(CVPixelBufferRef)foregroundPixelBuffer andBackgroundSourceBuffer:(CVPixelBufferRef)backgroundPixelBuffer forTweenFactor:(float)tween
{
    [self doesNotRecognizeSelector:_cmd];
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Это означает, что вся ответственность за реализацию рендеринга перекладывается
на классы наследники. Если посмотреть код этого метода в классах наследниках, то
мы увидим, что он во многом отличается. Так как логика эффекта перехода
заключается не только в шейдере, но и во предварительных настройках обработки
текстур перед передачей их на GPU.

В начале статьи упоминается о доработке этого семпла и некоего упрощения.
Начинается оно с методов переопределения пропертей:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- (NSDictionary *)sourcePixelBufferAttributes
{
    return @{ (NSString *)kCVPixelBufferPixelFormatTypeKey : [NSNumber numberWithUnsignedInt:kCVPixelFormatType_32BGRA],
(NSString*)kCVPixelBufferOpenGLESCompatibilityKey : [NSNumber numberWithBool:YES]};
}

- (NSDictionary *)requiredPixelBufferAttributesForRenderContext
{
    return @{ (NSString *)kCVPixelBufferPixelFormatTypeKey : [NSNumber numberWithUnsignedInt:kCVPixelFormatType_32BGRA],
(NSString*)kCVPixelBufferOpenGLESCompatibilityKey : [NSNumber numberWithBool:YES]};
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Для упрощения работы мы переходим с двупланарных буферов на буферы в RGB
формате. Что это означает? В двупланарных буферах мы имеем по две текстуры,
которые накладываются друг на друга. В RGB – мы имеем одну текстуру. Это
упрощает подход к реализации эффектов и позволяет всю реализацию обработки
текстур перенести на шейдеры. Таким образом, мы получим универсальный код
подготовки работы с OpenGL. А все разнообразие эфектов будет заключатся в
подставлении кода шейдера.

Этот подход был реализован в семпле VideoTransitionsSample. Большая часть кода
взята с проекта Apple с добавлением новой логики рендеринга. Если посмотреть на
обновленный APLOpenGLRenderer, то первым, что бросается в глаза, это полная
реализация метода renderPixelBuffer:
usingForegroundSourceBuffer:andBackgroundSourceBuffer:forTweenFactor:. При этом
в классах наследниках он не переопределяется. Еще одним изменением стал новый
метод fragmentShaderString, метод который подставляет код шейдера. Если в
наследнике переопределить это метод и вернуть новый код шейдера, то в коде
базового класса будет применятся уже новый шейдер, что позволяет легко добавлять
новые эффекты. Как например в классе VTSStarWipeRenderer, в котором кроме
статической строки с кодом для шейдера и переопределением нового метода, больше
ничего нет, так как ничего большего и не требуется.

Исходный код данного семпла можно найти на GitHub по ссылке
<https://github.com/iQueSoft/iOSDemo_VideoTransitionsSample> .
