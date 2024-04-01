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
                  <v-btn :icon="image.isSelected ? 'mdi-download-box' : 'mdi-download-outline'" size="large" :class="image.isSelected ? 'bg-green-accent-3' : 'bg-white'"/>
                </v-img>
              </v-item>
            </v-col>
          </v-row>
        </v-item-group>
      </v-container>
    </v-card>
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

    console.log(images);
    const principalObject: IImageObject = {
      name: keyword,
      images: [],
    };

    for (let j = 0; j < images.items.length; j++) {
      const imageUrl = images.items[j].link;
      principalObject.images.push({
        url: imageUrl,
        isSelected: false,
      });
    }

    searchedImages.value.push(principalObject);
  }
};

const downloadSelectedImages = async () => {
  try {
    for (let i = 0; i < searchedImages.value.length; i++) {
    const imgObj = searchedImages.value[i];

    for (let j = 0; j < imgObj.images.length; j++) {
      const imgElem = imgObj.images[j];

      if (imgElem.isSelected)
        await downloadImage(imgElem.url, `${imgObj.name}-${j}.jpg`)
    }
  }
  } catch (error) {
    console.log(error)
  }
};

const clearImages = () => {
  searchedImages.value = [];
  keywords.value = "";
};

async function downloadImage(url:string, title:string) {
    const image = await fetch(url);
    const imageURL = URL.createObjectURL(await image.blob());

    const link = document.createElement("a");
    link.href = imageURL;
    link.download = title;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
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
