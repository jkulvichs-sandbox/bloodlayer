<template>
  <audio style="display: none" ref="audio"></audio>
  <div class="xaudio">
    <div class="bars">
      <div
          v-for="(bar, i) in scaledBars"
          :key="i"
          :style="{ height: `${bar * 100}%` }"
          class="bar"
      ></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "XAudio",

  props: {
    // Ссылка на аудио файл
    src: {type: String, default: null}
  },

  data() {
    return {
      // Размеры баров отображения в условных единицах
      bars: [],
      // Части декодированного аудио
      audioParts: [],
    }
  },

  mounted() {
    this.startFetching()

    setTimeout(() => {
      let cursor = 0
      let stopInterval = 0
      stopInterval = setInterval(() => {
        const duration = this.audioDuration()
        this.bars.push(this.audioSampleLoud(cursor * 5000, 1000))
        cursor++
        if (cursor >= duration - 1) {
          clearInterval(stopInterval)
        }
      }, 100)
    }, 1000)
  },

  methods: {
    // Начинает загрузку и обработку аудио
    // Возвращает контекст отмены операции
    startFetching(src = './albums/Slaughter to Prevail - ZAVALI EBALO.mp3') {
      // Очищаем массив частей декодированных аудио
      this.audioParts = []

      // Управление отменой операций загрузки и обработки
      const abort = new AbortController();

      // Начинаем асинхронную загрузку и обработку аудио
      (async () => {
        // Начинаем загрузку ресурса, передавая в параметры возможный сигнал отмены
        const resp = await fetch(src, {
          signal: abort.signal,
        })
        // Создаём объект, способный обрабатывать аудио
        const ctx = new AudioContext()
        // Получаем объект, читающий данные из загружаемого потока
        const reader = resp.body.getReader()
        while (!abort.signal.aborted) {
          // Получаем флаг конца загрузки и часть загруженных данных
          const {done, value} = await reader.read()
          // Останавливаем чтение загружаемых данных, если достигнут конец потока
          if (done) break
          // Декодирование и добавление декодированного куска в декодированные части
          const audioPart = await ctx.decodeAudioData(value.buffer)
          // Добавление аудио участка в массив
          this.audioParts.push(audioPart)
        }
      })()

      // Возвращаем функцию отмены операции
      return abort
    },

    // ==========[ МЕТОДЫ АНАЛИЗА АУДИО ]========== //

    // Получить продолжительность загруженного аудио в миллисекундах
    audioDuration() {
      // Сохраняем ссылку на массив частей аудио
      const parts = this.audioParts
      // Идём по всем частям и суммируем их длину
      let duration = 0
      for (let i = 0; i < parts.length; i++) {
        duration += parts[i].duration
      }
      if (duration === 0) debugger
      return duration * 1000
    },

    // Получить общую громкость участка аудио
    // В качестве параметров позиция в миллисекундах
    audioSampleLoud(start, length = 1000, sampleRate = 44100) {
      // Сохраняем продолжительность аудио в мс
      const duration = this.audioDuration()

      // Проверка на выход за диапазон времени аудио
      if (start + length >= duration)
        throw new Error(`audioSampleLoud: out of range audio duration, got last pos: ${start + length}, max: ${duration - start - length - 1}`)

      // sampleRate в миллисекунду
      const sRateMs = sampleRate / 1000
      const throttle = 10

      // Получаем громкость на всех подучастках с шагом в 1/10 секунды
      const louds = []
      for (let i = (start) * sRateMs; i < (start + length) * sRateMs; i += sRateMs * throttle) {
        louds.push(this.audioQuantSampleLoud(i, sRateMs))
      }

      // Возвращаем среднюю громкость участка
      return louds.reduce((acc, val) => acc + val) / louds.length
    },

    // Возвращает громкость участка в условных единицах между позициями двух квантов
    audioQuantSampleLoud(start, length = 44.1) {
      // Инициализация пиковых значений максимальными противоположными величинами
      let min = 1
      let max = -1
      // Пропуск подсчёта квантов для производительности
      const throtle = 20
      // Итерация и поиск актуальных пиковых значений
      for (let i = start | 0; i < (start | 0) + (length | 0); i += throtle) {
        const quant = this.audioQuantAt(i)
        if (quant < min) min = quant
        if (quant > max) max = quant
      }
      // Высчитываем разницу и приводим к единице
      let loud = (Math.max(max, min) - Math.min(min, max)) / 2
      if (loud === 1) loud = .5
      return loud
    },

    // Получить общую продолжительность аудио в квантах
    audioQuantsLength() {
      let length = 0
      this.audioParts.forEach(part => length += part.length)
      return length
    },

    // Получить квант аудио по индексу
    audioQuantAt(pos) {
      // Сохраняем общую длину аудио в квантах
      const length = this.audioQuantsLength()
      // Сохраняем ссылку на массив частей аудио
      const parts = this.audioParts

      // Проверяем, что требуемая позиция не вышела за диапазон общей длины аудио
      if (pos < 0 || pos >= length)
        throw new Error(`audioAt: pos overflow, got: ${pos}, expected between [0, ${length}`)

      // Возвращает квант смёрженной волны для всех каналов
      const getMergedQuant = (buffer, pos) => {
        if (pos >= buffer.length)
          pos = buffer.length - 1
          // throw new Error(`audioQuantAt: getMergedQuant: pos out of range, got: ${pos}, max: ${buffer.length}`)

        let accumulator = 0
        // Суммируем значение в этой позиции всех каналов
        for (let i = 0; i < buffer.numberOfChannels; i++) {
          accumulator += buffer.getChannelData(i)[pos]
        }
        // Делим сумму на количество каналов для получения усредненного значения кванта
        const quant = accumulator / buffer.numberOfChannels
        return quant
      }

      // Сумма смещений по длинам кусков
      let cursorOffset = 0
      // Итерируемся по кускам, пока не найдем кусок в нужной позиции
      for (let ipart = 0; ipart < parts.length; ipart++) {
        // Проверка перед следущим условием чтобы не выйти за пределы массива частей аудио
        if (ipart < parts.length - 1) {
          // Если требуемая позиция более текущего смещения и менее следующего - кусок найден
          if (pos >= cursorOffset && pos < cursorOffset + parts[ipart + 1].length) {
            // Ищем значение конкретной кванты в части аудио
            return getMergedQuant(parts[ipart], pos - cursorOffset)
          }
        } else {
          // Если сработало это условие - значит мы добрались до последнего блока аудио
          // Ищем позицию в нём
          return getMergedQuant(parts[ipart], pos - cursorOffset)
        }
        cursorOffset += parts[ipart].length
      }
    },
  },

  computed: {
    // Бары в относительных друг от другах велечинах от 0 до 1
    scaledBars() {
      // Размер самого большого бара
      let max = 0
      // Поиск самого большого бара
      this.bars.forEach(bar => {
        if (bar > max) max = bar
      })
      // Формирование нового массива относительных размеров баров
      return this.bars.map(bar => bar / max).map(bar => Math.max(bar, .1))
    },
  },

// test1() {
//   (async () => {
//     // Запрос на загрузку аудио
//     const resp = await fetch('./albums/Slaughter to Prevail - ZAVALI EBALO.mp3')
//     const respLength = +resp.headers.get('content-length')
//
//     // Контекст обработчика аудио
//     const audioCtx = new AudioContext()
//     const audioParts = []
//
//     // Чтение частей аудио и добавление в буффер
//     const reader = resp.body.getReader()
//     while (true) {
//       const {done, value} = await reader.read()
//       if (done) break
//       // Декодирование аудио и добавление куска
//       const decodedAudio = await audioCtx.decodeAudioData(value.buffer)
//
//       const source = audioCtx.createBufferSource()
//       source.buffer = decodedAudio
//       source.connect(audioCtx.destination)
//       source.start(0)
//     }
//
//   })()
// },
//
// test2() {
//   // (async () => {
//   //   const audio = new Audio('./albums/Slaughter to Prevail - ZAVALI EBALO.mp3')
//   //   const ctx = new AudioContext()
//   //   const track = ctx.createMediaElementSource(audio)
//   //   const a = ctx.createMediaStreamSource(audio)
//   //   console.log(track)
//   // })()
// },
//
// Получение паршеного буфера аудио
// (async () => {
//   const resp = await fetch('./albums/Slaughter to Prevail - ZAVALI EBALO.mp3')
//   const audioBlob = await resp.blob()
//   const audioDataBuffer = await audioBlob.arrayBuffer()
//   const audioCtx = new AudioContext()
//   const buffer = await audioCtx.decodeAudioData(audioDataBuffer)
//   console.log(buffer)
// })()
//
// Получить смёрженный канал буфера аудио
// (async () => {
//   const channelsCount = buffer.numberOfChannels
//   const mainChannel = buffer.getChannelData(0)
//   const mergedChannel = new Float32Array(mainChannel.length)
//   for (let i = 0; i < mainChannel.length; i++) {
//     for (let channelId = 0; channelId < channelsCount; channelId++) {
//       mergedChannel[i] += buffer.getChannelData(channelId)[i]
//     }
//     mergedChannel[i] /= channelsCount
//   }
//   console.log(mergedChannel)
// })()
//
// (async () => {
//   const audio = document.createElement('audio')
//   audio.src = './albums/Slaughter to Prevail - ZAVALI EBALO.mp3'
//   const reader = new ReadableStream(audio.captureStream().getAudioTracks()[0]).getReader()
//   console.log(await reader.read())
// })()
//
// (async () => {
//   const audio = new Audio('./albums/Slaughter to Prevail - ZAVALI EBALO.mp3')
//   audio.addEventListener('canplaythrough', () => {
//     const mediaTrack = audio.captureStream()
//     const recorder = new MediaRecorder(mediaTrack)
//     recorder.ondataavailable = ev => {
//       console.log(ev.data)
//     }
//     recorder.start(1000)
//   })
// })()

}
</script>

<style lang="sass" scoped>
.xaudio
  width: 100%
  height: 50px
  background: #212121
  padding: 10px

  .bars
    display: flex
    justify-content: center
    align-items: center
    width: 100%
    height: 100%

    .bar
      width: 5px
      background: #FFFFFF
      border-radius: 5px
      transition: .1s ease-out
      animation: .3s ease-out both bar-anim

      &:not(:last-child)
        margin-right: 2.5px

@keyframes bar-anim
  0%
    filter: opacity(0)
  100%
    filter: opacity(1)
</style>
