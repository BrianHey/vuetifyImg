<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-spacer></v-spacer>
    </v-app-bar>

    <v-container>
      <v-main>
        <v-text-field v-model="nombre" label="nombre"></v-text-field>
        <v-text-field v-model="edad" label="edad"></v-text-field>
        <v-file-input
          @change="previewImage"
          accept="image/*"
          label="File input"
        ></v-file-input>
        <v-btn depressed color="primary" @click="upload">
          Subir al storage
        </v-btn>
        <div v-show="loading">
          <h3>Registrando persona</h3>
          <v-progress-linear
            color="deep-purple accent-4"
            indeterminate
            rounded
            height="6"
          ></v-progress-linear>
        </div>
        <v-card
          v-for="(persona, i) in personas"
          :key="i"
          class="mx-auto"
          max-width="344"
        >
          <v-img :src="persona.data.imgSrc" height="200px"></v-img>

          <v-card-title>
            {{ persona.data.name }}
          </v-card-title>

          <v-card-subtitle>
            {{ persona.data.edad }}
          </v-card-subtitle>
        </v-card>
      </v-main>
    </v-container>
  </v-app>
</template>

<script>
import firebase from "firebase";
export default {
  name: "App",
  mounted() {
    firebase
      .firestore()
      .collection("personas")
      .onSnapshot((snapshot) => {
        this.personas = [];
        snapshot.forEach((doc) => {
          this.personas.push({
            id: doc.id,
            data: doc.data(),
          });
        });
      });
  },
  methods: {
    previewImage(file) {
      console.log(file);
      this.imagen = file;
    },
    upload() {
      this.loading = true;
      firebase
        .storage()
        .ref(`/imagenes/${this.imagen.name}`)
        .put(this.imagen)
        .then((snapshot) => {
          snapshot.ref.getDownloadURL().then((url) => {
            console.log(url);
            this.url = url;

            firebase
              .firestore()
              .collection("personas")
              .add({
                name: this.nombre,
                edad: this.edad,
                imgSrc: this.url,
              })
              .then(() => {
                this.loading = false;
              });
          });
        });
    },
  },

  data: () => ({
    imagen: "",
    url: "",
    nombre: "",
    edad: "",
    personas: [],
    loading: false,
  }),
};
</script>
