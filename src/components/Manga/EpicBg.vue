<template>
  <div class="epic-bg">

    <img id="epicImg" class="epic-img" src="">
    <div class="gradient"></div>

    <div class="center-edit-bg">

      <label v-if="manga.edit && !epicPreview && !uploadingEpicImg" for="img-upload" class="edit-bg">Change background image
        <input id="img-upload" type="file" @change="previewImg">
      </label>
      
      <div v-if="epicPreview" class="epic-keep">
        <p class="text-center mb-2">Keep image?</p>
        <div class="grid grid-cols-2 gap-4">
          <div @click="epicYES()" class="text-center yes-no">YES</div>
          <div @click="epicNO()" class="text-center yes-no">NO</div>
        </div>
      </div>

      <div class="epic-title text-center"> <b>MANGA: </b> <b v-if="$store.state.tmangaTotal==0 || $store.state.mangaTotal==$store.state.tmangaTotal">{{$store.state.mangaTotal}}</b> <b v-else>{{$store.state.tmangaTotal}}</b> </div>
      
      <div v-if="uploadingEpicImg" class="ultra-center">
        <Loading msg="Uploading image"/>
      </div>
    </div>

  </div>
</template>

<script>
import manga from "@/manga";
import data from "@/data";
import { doc, db, updateDoc, collection, getDocs } from "@/firebase";
import { storage, ref, uploadBytes, deleteObject, getDownloadURL, listAll} from "@/firebase";
import Loading from '@/components/Loading.vue';

export default {
  name: "EpicBg",
  components: {
    Loading,
  },
  data() {
    return {
      //EPICIMG
      epicImg: null,
      epicPreview: false,
      r_epicImg: null,
      uploadingEpicImg: false,

      manga,
      data,
    }
  },
  mounted() {
    this.loadEpicImg();
  },
  methods: {
    dummy() {},

    async loadEpicImg() {
      try {
        let querySnapshot = await getDocs(collection(db, "users"));
        querySnapshot.forEach((doc) => {
          if (data.email === `${doc.data().email}`) {
            manga.epicImg = `${doc.data().epicImg}`;
          }
        })
        let img = document.getElementById("epicImg");
        img.src = manga.epicImg;
      }
      catch (e) {
        console.error(e);
      }
    },

    previewImg(event) {
      let img = document.getElementById("epicImg");
      this.r_epicImg = img.src;
      this.epicImg = event.target.files[0]; 
      this.epicPreview = true;
      img.src = URL.createObjectURL(event.target.files[0]);
    },

    epicNO() {
      this.epicPreview = false;
      let img = document.getElementById("epicImg");
      img.src = this.r_epicImg;
    },

    async epicYES() {
      try {
        this.uploadingEpicImg = true;
        this.epicPreview = false;
        let imageName = "images/" + "manga/" + data.username + "/" + this.epicImg.name;
        const storageRef = ref(storage, imageName);

        let find = "images/" + "manga/" + data.username;
        let listRef = ref(storage, find);

        let imageRef = "";
        let list = await listAll(listRef);
        list.items.forEach((itemRef) => {
          imageRef = ref(storage, "images/" + "manga/" + data.username + "/" + itemRef.name);
        });

        await uploadBytes(storageRef, this.epicImg);
        console.log("Image " + imageName + " uploaded!");

        let url = await getDownloadURL(ref(storage, imageName))
        const g = doc(db, "users", data.id);
        await updateDoc(g, {
          epicImg: url,
        })
        console.log("Deleting previous image...");
        await deleteObject(imageRef);
        console.log("Image " + imageRef + " deleted!");
        this.uploadingEpicImg = false;
      }
      catch (e) {
        console.error(e);
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.ultra-center {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.epic-title {
  font-size: 82px;
  color: white;
  text-shadow: 4px 2px black;
}
.epic-bg {
  height: 300px;
  display: flex;
  overflow: hidden;
  position: relative;
  width: 100%;
  overflow: hidden;
  .epic-img {
    position: absolute;
    width: 100%;
    left:50%;
    top:50%;
    transform:translate(-50%,-50%);
    pointer-events: none;
  }
  .gradient  {
    z-index: 10;
    width: 100%;
    height: 100%;
    background-image: linear-gradient(180deg, #0b162200, #0b162273);
  }
  .center-edit-bg {
    position: absolute;
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    .edit-bg {
      transition: .5s;
      position: absolute;
      background-color: rgba(35,38,67,.5);
      box-shadow: 0px 0px 8px -2px #0b1622b9;
      padding: 4px 16px 4px 16px;
      border-radius: 16px;
      color: #EDF1F5;
      z-index: 20;
      &:hover {
        box-shadow: 0px 0px 10px -2px #0b1622;
        background-color: #151F2E;
        color: #9FADBD;
        cursor: pointer;
      }
    }
    .epic-keep {
      z-index: 150;
      background-color: rgba(35,38,67,.5);
      box-shadow: 0px 0px 8px -2px #0b1622b9;
      padding: 8px 48px 8px 48px;
      border-radius: 48px;
      color: #EDF1F5;
      .yes-no {
        background-color: rgba(35,38,67,.5);
        box-shadow: 0px 0px 8px -2px #0b1622b9;
        padding: 2px 16px 2px 16px;
        border-radius: 16px;
        color: #EDF1F5;
        z-index: 150;
        &:hover {
          box-shadow: 0px 0px 10px -2px #0b1622;
          background-color: #3DB4F2;
          font-weight: bold;
          cursor: pointer;
        }
      }
      &:hover {
        box-shadow: 0px 0px 10px -2px #0b1622;
        background-color: #151F2E;
        color: #9FADBD;
      }
    }
  }
}
</style>