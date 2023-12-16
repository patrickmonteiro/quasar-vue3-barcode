<template>
  <q-page>
    <p class="text-h6 text-center">Barcode Reader QuaggaJs</p>

    <div class="text-center">
      <q-btn
        label="Iniciar Leitura"
        color="primary"
        icon="mdi-barcode-scan"
        size="lg"
        @click="initReader"
      />
    </div>
    <p
      v-if="code"
      class="text-h6 text-center q-mt-lg">
      Código Lido: {{ code }}
      <q-btn
        icon="mdi-content-copy"
        @click="copyCode"
        size="md"
        dense
        color="blue-9"
      />
    </p>
    <div
      class="text-center"
      v-show="cameraStatus"
      id="reader"></div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from 'vue'
import Quagga from 'quagga'
import { copyToClipboard, useQuasar } from 'quasar'

export default defineComponent({
  name: 'IndexPage',
  setup () {
    const code = ref('')
    const cameraStatus = ref(false)
    const q$ = useQuasar()
    const initReader = () => {
      cameraStatus.value = true
      Quagga.init({
        inputStream: {
          name: 'Live',
          type: 'LiveStream',
          target: document.querySelector('#reader')
        },
        decoder: {
          readers: ['ean_reader']
        }
      }, function (err) {
        if (err) {
          console.log(err)
          return
        }
        console.log('Initialization finished. Ready to start')
        Quagga.start()
        Quagga.onDetected((data) => {
          onDetected(data)
        })
      })
    }

    const stopReader = () => {
      cameraStatus.value = false
      Quagga.stop()
    }

    const onDetected = (data) => {
      code.value = data.codeResult.code
      stopReader()
      console.log(data)
    }

    const copyCode = () => {
      copyToClipboard(code.value)
        .then(() => {
          q$.notify({
            message: 'Código de barras copiado com sucesso!',
            type: 'positive'
          })
        })
        .catch(() => {
          q$.notify({
            message: 'Erro ao copiar Código de barras!',
            type: 'negative'
          })
        })
    }
    return {
      initReader,
      stopReader,
      cameraStatus,
      code,
      copyCode
    }
  }
})
</script>
