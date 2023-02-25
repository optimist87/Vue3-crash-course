<template>
  <div>
    <h1>Картинки</h1>
    <div class="select-wrapper row">
      <my-select
        v-model="selectedSortCount"
        :options="sortOptionsCount"
      />
      <my-select
        v-if="breadList.length > 0"
        v-model="breadId"
        :options="breadListForSelect"
      />
    </div>
    <h2>{{ currentBreadName }}</h2>
    <div class="row">
      <p v-if="currentBread?.life_span"><strong>Годы жизни:</strong> {{ currentBread?.life_span }} лет</p>
      <p v-if="currentBread?.metric"><strong>Вес:</strong> {{ currentBread?.weight.metric }} кг.</p>
      <p v-if="currentBread?.origin"><strong>Страна:</strong> {{ currentBread?.origin }}</p>
      <p v-if="currentBread?.temperament"><strong>Темперамент:</strong> {{ currentBread?.temperament }}</p>
    </div>
    <my-button
      @click="loadPictures"
      :disabled="isLoadingPictures"
      class="row"
    >
      {{ isLoadingPictures ? "Загрузка" : "Загрузить других котиков" }}
    </my-button>
    <div class="wrapper">
      <p v-if="isLoadingPictures">Загрузка...</p>
      <div v-else v-for="item in pictures" class="wrapper-item">
        <h4>{{ item.id }}</h4>
        <img :src="item.url" :alt="item.id" class="item" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import MySelect from "@/components/UI/MySelect";
import { apiUrl, apiUrlBreads } from "@/config"

export default {
  components: {
    MySelect,
  },
  data() {
    return {
      isLoadingPictures: false,
      isLoadingBreads: true,
      pictures: [],
      breadList: [],
      selectedSortCount: "10",
      breadId: "All",
      sortOptionsCount: [
        {
          name: "10",
          value: "10",
        },
        {
          name: "20",
          value: "20",
        },
        {
          name: "30",
          value: "30",
        },
        {
          name: "50",
          value: "50",
        },
      ]
    }
  },
  methods: {
    async loadPictures () {
      this.isLoadingPictures = true
      const { data } = await axios(apiUrl, {
        params: {
          limit: this.selectedSortCount,
          breed_ids: this.breadId === "All" ? undefined : this.breadId,
        },
        headers: {
          "x-api-key": "live_BjwApRdLlKEcPrk56E00IIYP1F1FqiLA0nztZInv4Q3T407YATCioRQh7OOWE4HV",
        }
      })
      this.pictures = data
      this.isLoadingPictures = false
    },
    async loadSortedBreads () {
      const { data } = await axios(apiUrlBreads, {
        headers: {
          "x-api-key": "live_BjwApRdLlKEcPrk56E00IIYP1F1FqiLA0nztZInv4Q3T407YATCioRQh7OOWE4HV",
        }
      })
      const reducedBreadList = data.reduce((acc, item) => {
        return [
          ...acc,
          {
            name: item.name,
            value: item.id,
            ...item,
          }
        ]
      }, [])
      this.breadList = reducedBreadList
      this.isLoadingBreads = false
      this.loadPictures()
    }
  },
  mounted() {
    this.loadSortedBreads()
  },
  watch: {
    selectedSortCount() {
      this.loadPictures()
    },
    breadId() {
      this.loadPictures()
    }
  },
  computed: {
    currentBread () {
      return [...this.breadList].find(item => item.id === this.breadId)
    },
    currentBreadName () {
      return this.currentBread?.name || "Котики"
    },
    breadListForSelect () {
      return [
        {
          name: "All",
          value: "All",
        },
        ...this.breadList,
      ]
    }
  }
}
</script>

<style scoped>
h1, h2 {
  margin-bottom: 20px;
}
.row {
  margin-bottom: 20px;
}
.select-wrapper {
  display: flex;
  gap: 10px;
}
.wrapper {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  align-items: flex-start;
}
.wrapper-item {
  max-width: 10%;
  padding: 5px;
}
.item {
  width: 100%;
}
</style>
