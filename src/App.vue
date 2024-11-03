<template>
  <LoaderComponent v-if="downloading || copying" :message="downloading ? 'Descargando...' : 'Copiando...'" />

  <v-sheet
    class="mx-auto principalSheet bg-grey-darken-3"
    elevation="8"
    max-width="1200"
  >

    <div class="titleContainer bg-grey-darken-3">

      <h1>
        Easy Images
        <v-icon>mdi-image-multiple</v-icon>
      </h1>

      <v-container class="subContainer">
        <v-btn
        class="downloadBtn titleBtn btn"
        color="pink"
        prepend-icon="mdi-download"
        @click.prevent="downloadSelectedImages"
      >
        <template v-slot:prepend>
          <v-icon color="black"></v-icon>
        </template>
        Descargar
      </v-btn>

      <v-btn
        class="downloadBtn titleBtn btn"
        color="pink"
        prepend-icon="mdi-content-copy"
        @click.prevent="copySelectedImages"
      >
        <template v-slot:prepend>
          <v-icon color="black"></v-icon>
        </template>
        <v-tooltip
        activator="parent"
        location="top"
      >Tomar en cuenta que solo se copia una imagen a la vez</v-tooltip>
        Copiar
      </v-btn>

      <v-btn
      class="clearBtn titleBtn btn"
      color="pink"
      prepend-icon="mdi-backspace-outline"
      @click.prevent="clearImages"
    >
      <template v-slot:prepend>
        <v-icon color="black"></v-icon>
      </template>
      Limpiar
    </v-btn>
      </v-container>
      
    </div>

    <v-form v-model="form" @submit.prevent="onSubmit" class="mb-4">
      <v-text-field
        v-model="keywords"
        :readonly="loading"
        append-icon="mdi-magnify"
        class="mb-2"
        clearable
        label="Ingrese sus palabras mediante comas (,) si desea buscar más de una imagen a la vez"
      ></v-text-field>
      <v-btn
        class="btnBuscar btn"
        :disabled="keywords === '' || keywords === null"
        :loading="loading"
        color="success"
        type="submit"
        variant="elevated"
      >
        Buscar
      </v-btn>
    </v-form>
    <!-- NUEVO -->
    <v-card
      class="mx-auto bg-grey-darken-3"
      v-if="searchedImages.length > 0"
    >
      <v-container class="pa-1" :fluid="true">
        <v-item-group
          multiple
          v-for="(imageObject, principalIndex) in searchedImages"
          :key="imageObject.name"
        >
        <h2>{{ imageObject.name.toUpperCase() }}</h2>
          <v-row>
            <v-col
              v-for="image in imageObject.images"
              :key="image.url"
              cols="12"
              md="4"
            >
              <v-item>
                <v-img
                  :src="image.url"
                  :class="image.url.endsWith('.png') ? 'bg-grey-lighten-4' : 'text-right pa-2'"
                  :height="250"
                  @click="toggleSelection(principalIndex, image.url)"
                >
                  <v-btn :icon="image.isSelected ? 'mdi-check-circle' : 'mdi-check-circle-outline'" size="large" :class="image.isSelected ? 'bg-green-accent-3' : 'bg-white'"/>
                </v-img>
              </v-item>
            </v-col>
          </v-row>
        </v-item-group>
      </v-container>
    </v-card>
  </v-sheet>

  <v-snackbar
      :timeout="2000"
      color="success"
      variant="outlined"
      v-model="snackbar"
      location="center"
    >
      Imágenes copiadas al portapapeles
      <template v-slot:actions>
        <v-btn
          @click="snackbar = false"
        >
          Cerrar
        </v-btn>
      </template>
    </v-snackbar>
</template>

<script setup lang="ts">
import { ref, Ref } from "vue";
import { IImageObject } from "@/interfaces/IImageObject";
import LoaderComponent from "@/components/LoaderComponent.vue";

const keywords: Ref<""> = ref("");

const form: Ref<boolean> = ref(false);
const loading: Ref<boolean> = ref(false);
const downloading: Ref<boolean> = ref(false);
const copying: Ref<boolean> = ref(false);
const snackbar: Ref<boolean> = ref(false);

const searchedImages: Ref<Array<IImageObject>> = ref([]);

const onSubmit = async () => {
  try {
    if (!form.value) return;

    const words = keywords.value.split(",");
    const trimmedWords = words.map((word) => word.trim());
    loading.value = true;
    await findImagesFromGoogle(trimmedWords);
    loading.value = false;
  } catch (error) {
    console.error(error);
  }
};

const findImagesFromGoogle = async (keywords: Array<string>) => {
  const apiKey = process.env.VUE_APP_GOOGLE_SEARCH_API_KEY;
  const cx = process.env.VUE_APP_GOOGLE_SEARCH_ENGINE_ID;
  const apiUrl = `https://www.googleapis.com/customsearch/v1?key=${apiKey}&cx=${cx}&searchType=image&lr=lang_es&q=`;

  for (let i = 0; i < keywords.length; i++) {
    const keyword = keywords[i];
    const response = await fetch(`${apiUrl}${encodeURIComponent(keyword)}`);

    const images = await response.json();

    // console.log(images);
    const principalObject: IImageObject = {
      name: keyword,
      images: [],
    };

    for (let j = 0; j < images.items.length; j++) {
      const imageUrl = images.items[j].link;

      const imageExtension = images.items[j].mime.split('/')[1];

      const imageTitle = images.items[j].title + '.' + imageExtension;
      
      principalObject.images.push({
        url: imageUrl,
        isSelected: false,
        title: imageTitle,
      });
    }

    searchedImages.value.push(principalObject);
  }
};

const downloadSelectedImages = async () => {
  try {
    for (let i = 0; i < searchedImages.value.length; i++) {
    const imgObj = searchedImages.value[i];
      
    downloading.value = true;
    for (let j = 0; j < imgObj.images.length; j++) {
      const imgElem = imgObj.images[j];

      if (imgElem.isSelected)
        await downloadImage(imgElem.url, imgElem.title);
    }

    downloading.value = false;
  }
  } catch (error) {
    console.log(error)
    downloading.value = false;
  }
};

const copySelectedImages = async () => {
  try {
    for (let i = 0; i < searchedImages.value.length; i++) {
    const imgObj = searchedImages.value[i];
      
    copying.value = true;
    for (let j = 0; j < imgObj.images.length; j++) {
      const imgElem = imgObj.images[j];

      if (imgElem.isSelected)
        await copyImage(imgElem.url);
    }

    copying.value = false;
    snackbar.value = true;
  }
  } catch (error) {
    console.log(error)
    copying.value = false;
    snackbar.value = false;
  }
};

const clearImages = () => {
  searchedImages.value = [];
  keywords.value = "";
};

async function downloadImage(url:string, title:string) {
  const proxyUrl = process.env.VUE_APP_PROXY_URL;

const response = await fetch(proxyUrl + encodeURIComponent(url), {
  method: 'GET',
  headers: {
    'x-api-key': process.env.VUE_APP_PROXY_API_KEY // API Key en el header personalizado
  }
});

if (!response.ok) {
    throw new Error('Failed to fetch resource');
}

const blob = await response.blob();
const imageURL = URL.createObjectURL(blob);

const link = document.createElement("a");
link.href = imageURL;
link.download = title;
document.body.appendChild(link);
link.click();
document.body.removeChild(link);
}

async function copyImage(url:string) {
  const proxyUrl = process.env.VUE_APP_PROXY_URL;

  const response = await fetch(proxyUrl + encodeURIComponent(url), {
    method: 'GET',
    headers: {
      'x-api-key': process.env.VUE_APP_PROXY_API_KEY // API Key en el header personalizado
    }
  });

  if (!response.ok) {
      throw new Error('Failed to fetch resource');
  }

  const blob = await response.blob();
  // Crear una imagen desde el Blob
  const img = document.createElement('img');
  img.src = URL.createObjectURL(blob);

  img.onload = async () => {
    // Crear un canvas y dibujar la imagen en él
    const canvas = document.createElement('canvas');
    canvas.width = img.width;
    canvas.height = img.height;

    const ctx = canvas.getContext('2d');

    if (ctx !== null) {
      ctx.drawImage(img, 0, 0);
    }

    // Convertir el canvas a Blob en formato PNG
    canvas.toBlob(async (pngBlob) => {
      if (pngBlob) {
        const clipboardItem = new ClipboardItem({ 'image/png': pngBlob });
        try {
          await navigator.clipboard.write([clipboardItem]);
          console.log("Imagen copiada al portapapeles en formato PNG");
        } catch (error) {
          console.error("Error al copiar la imagen:", error);
        }
      }
    }, 'image/png');
    
    // Liberar URL de la imagen original
    URL.revokeObjectURL(img.src);
  };
}

// async function copyImages(urls: Array<string>) {
//   const canvas = document.createElement('canvas');
//   const ctx = canvas.getContext('2d');

//   // Primero cargamos todas las imágenes y calculamos el tamaño del canvas
//   const images = await Promise.all(urls.map(async (url) => {
//     const proxyUrl = process.env.VUE_APP_PROXY_URL;

//     const response = await fetch(proxyUrl + encodeURIComponent(url), {
//       method: 'GET',
//       headers: {
//         'x-api-key': process.env.VUE_APP_PROXY_API_KEY // API Key en el header personalizado
//       }
//     });

//     if (!response.ok) {
//         throw new Error('Failed to fetch resource');
//     }

//     const blob = await response.blob();
//     const img = new Image();
//     img.src = URL.createObjectURL(blob);
//     await img.decode(); // Esperamos a que la imagen cargue completamente
//     return img;
//   }));

//   // Establece el tamaño del canvas en función de las imágenes cargadas
//   canvas.width = Math.max(...images.map(img => img.width));
//   canvas.height = images.reduce((sum, img) => sum + img.height, 0);

//   // Dibuja cada imagen en el canvas una debajo de la otra

//   if (ctx !== null) {
//     let yOffset = 0;
//     images.forEach(img => {
//       ctx.drawImage(img, 0, yOffset);
//       yOffset += img.height;
//     });

//     // Convierte el canvas a PNG y copia al portapapeles
//     canvas.toBlob(async (pngBlob) => {
//       const clipboardItem = new ClipboardItem({ 'image/png': pngBlob });
//       try {
//         await navigator.clipboard.write([clipboardItem]);
//         console.log("Imágenes combinadas copiadas al portapapeles en formato PNG");
//       } catch (error) {
//         console.error("Error al copiar la imagen combinada:", error);
//       }
//     }, 'image/png');
//   }
// }

const toggleSelection = (index: number, url: string): void => {
  const isSelectedItem = isSelected(index, url);

  const currentImage = searchedImages.value[index];

  if (isSelectedItem) {
    currentImage.images.forEach((imgObj) => {
      if (imgObj.url === url) imgObj.isSelected = false;
    });
  } else {
    currentImage.images.forEach((imgObj) => {
      if (imgObj.url === url) imgObj.isSelected = true;
    });
  }
};

const isSelected = (index: number, url: string): boolean => {
  const currentImage = searchedImages.value[index];

  let selected = false;
  for (let i = 0; i < currentImage.images.length; i++) {
    const imgObj = currentImage.images[i];
    if (imgObj.url === url) {
      selected = imgObj.isSelected;
      break;
    }
  }
  return selected;
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background-color: #202123;
  color: #fff;
  padding-top: 60px;
  min-height: 100vh;
}

.titleContainer {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 1rem;
  padding: 0.5rem;
  position: sticky;
  top: 0px;
  z-index: 100;
}

.subContainer {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.titleBtn {
  width: 15%;
}

.btnBuscar {
  width: 20%;
}

.principalSheet {
  padding: 1rem;
}

.imageNotFound {
  width: 20%;
}

@media screen and (max-width: 768px) {
  /* .titleContainer {
    flex-direction: column;
    gap: 0.5rem;
  } */

  .subContainer {
    flex-direction: column;
    align-items: center; 
    justify-content: center;
  }

  h1 {
    order: -1;
  }

  .btn {
    width: 40%;
  }

  .slideGroup {
    padding: 0 !important;
  }
}
</style>
