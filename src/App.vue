<template>
  <v-sheet
    class="mx-auto principalSheet bg-grey-darken-3"
    elevation="8"
    max-width="1200"
  >
    <div class="titleContainer bg-grey-darken-3">
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
      <h1>
        Easy Images
        <v-icon>mdi-image-multiple</v-icon>
      </h1>

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

    <div v-for="(imageObject, index) in searchedImages" :key="imageObject.name">
      <h2>{{ imageObject.name.toUpperCase() }}</h2>
      <v-slide-group
        class="slideGroup"
        selected-class="bg-primary"
        multiple
        show-arrows
      >
        <v-slide-group-item
          v-slot="{ selectedClass }"
          v-if="imageObject.images.length > 0"
        >
          <v-card
            color="blue"
            :class="['ma-4', selectedClass]"
            height="200"
            width="200"
            @click="toggleSelection(index, url.url)"
            v-for="url in imageObject.images"
            :key="url.url"
          >
            <div class="d-flex fill-height align-center justify-center">
              <v-img :src="url.url" height="200px" cover class="cardImage">
              </v-img>

              <v-scale-transition>
                <v-icon
                  v-if="url.isSelected"
                  color="black"
                  size="48"
                  icon="mdi-check-bold"
                ></v-icon>
              </v-scale-transition>
            </div>
          </v-card>
        </v-slide-group-item>
        <v-slide-group-item v-else>
          <v-card class="mx-auto bg-blue-grey-lighten-4">
            <img src="@/assets/image-not-found.svg" class="imageNotFound" />
            <v-card-title class="text-h5">
              No se encontraron imágenes
            </v-card-title>
          </v-card>
        </v-slide-group-item>
      </v-slide-group>
    </div>
  </v-sheet>
</template>

<script setup lang="ts">
import { ref, Ref } from "vue";
import { IImageObject } from "@/interfaces/IImageObject";
const keywords: Ref<""> = ref("");

const form: Ref<boolean> = ref(false);
const loading: Ref<boolean> = ref(false);

const searchedImages: Ref<Array<IImageObject>> = ref([]);

const onSubmit = async () => {
  try {
    if (!form.value) return;

    const words = keywords.value.split(",");
    const trimmedWords = words.map((word) => word.trim());
    loading.value = true;
    await findImages(trimmedWords);
    loading.value = false;
  } catch (error) {
    console.error(error);
  }
};

const findImages = async (keywords: Array<string>) => {
  const apiKey = process.env.VUE_APP_ACCESS_KEY;
  const apiUrl = `https://pixabay.com/api/?key=${apiKey}&q=`;

  for (let i = 0; i < keywords.length; i++) {
    const keyword = keywords[i];
    const response = await fetch(`${apiUrl}${encodeURIComponent(keyword)}`);

    const images = await response.json();

    const principalObject: IImageObject = {
      name: keyword,
      images: [],
    };

    for (let j = 0; j < images.hits.length; j++) {
      const imageUrl = images.hits[j].webformatURL;
      principalObject.images.push({
        url: imageUrl,
        isSelected: false,
      });
    }

    searchedImages.value.push(principalObject);
  }
};

const downloadSelectedImages = async () => {
  for (let i = 0; i < searchedImages.value.length; i++) {
    const imgObj = searchedImages.value[i];

    for (let j = 0; j < imgObj.images.length; j++) {
      const imgElem = imgObj.images[j];

      if (imgElem.isSelected)
        await downloadImage(imgElem.url, `${imgObj.name}-${j}.jpg`);
    }
  }
};

const clearImages = () => {
  searchedImages.value = [];
  keywords.value = "";
};

async function downloadImage(photoUrl: string, nombreArchivo: string) {
  const photo = await fetch(photoUrl);
  // Get the photo blop
  const photoBlop = await photo.blob();

  const url = window.URL.createObjectURL(photoBlop);

  var a = document.createElement("a");
  a.style.display = "none";
  a.href = url;
  a.download = nombreArchivo;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);

  window.URL.revokeObjectURL(url);
}

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
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem;
  position: sticky;
  top: 0px;
  z-index: 100;
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
  .titleContainer {
    flex-direction: column;
    gap: 0.5rem;
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
