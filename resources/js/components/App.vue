<template>
  <div>
    <Nav />
    <div class="px-6">
        <h1 class="font-bold text-2xl text-gray-900 mt-2">Articles</h1>

        <form @submit.prevent="addArticle()" class="mt-3">
            <div>
                <input v-model="article.title" id="title" type="text" class="w-full rounded p-3 focus:outline-none" name="title" required placeholder="Title">
            </div>
            <div class="mt-2">
                <textarea v-model="article.body" id="body" class="w-full rounded p-3 focus:outline-none" name="body" required placeholder="body"></textarea>
            </div>
            <button class="bg-blue-500 text-white p-2 rounded-lg mt-2 w-full focus:outline-none" type="submit">Add Article</button>
        </form>

        <ul class="flex border rounded bg-white mt-6">
            <li :class="{'disabled': !pagination.prev_page_url}" :disabled="!pagination.prev_page_url">
                <a @click="fetchArticles(pagination.prev_page_url)" class="block text-blue-400 border-r px-3 py-2" href="#">Previous</a>
            </li>
            <li class="disabled">
                <a class="block text-blue-400 border-r px-3 py-2" href="#">page {{ pagination.current_page }} of {{ pagination.last_page }}</a>
            </li>
            <li :class="{'disabled': !pagination.next_page_url}" :disabled="!pagination.next_page_url">
                <a class="block text-blue-400 p-2" @click="fetchArticles(pagination.next_page_url)" href="#">Next</a>
            </li>
        </ul>

        <div
            v-for="(article, index) in articles['data']"
            :key="index"
            class="rounded shadow-sm px-6 py-4 bg-white mt-2 mb-2"
        >
            <div class="font-bold text-xl mb-2 text-gray-600">{{ article.data.title }}</div>
            <p class>{{ article.data.body }}</p>
            <button @click="editArticle(article.data)" class="bg-yellow-600 rounded-lg text-white w-full p-1 mt-2 focus:outline-none">Edit</button>
            <button @click="deleteArticle(article.data.id)" class="bg-red-600 rounded-lg text-white w-full p-1 mt-2 focus:outline-none">Delete</button>
        </div>
    </div>
  </div>
</template>

<script>
import Nav from "./Nav";

export default {
  name: "App",

  components: {
    Nav
  },

  data() {
    return {
      articles: [],
      article: {
        id: "",
        title: "",
        body: ""
      },
      article_id: null,
      pagination: {},
      edit: false
    };
  },

  created() {
    this.fetchArticles();
  },

  methods: {
    fetchArticles(page_url) {
      let vm = this;
      page_url = page_url || "/api/articles";
          axios.get(page_url)
            .then(res => {
                this.articles = res.data;
                vm.makePagination(res.data["meta"], res.data["links"]);
            })
            .catch(err => console.log(err));
    },

    makePagination(meta, links) {
      let pagination = {
        current_page: meta.current_page,
        last_page: meta.last_page,
        next_page_url: links.next,
        prev_page_url: links.prev
      };

      this.pagination = pagination;
    },

    deleteArticle(id) {
       if (confirm('Are You Sure?')) {
            axios.delete(`/api/articles/${id}`)
                .then(res => {
                    this.fetchArticles();
                    alert('Article Deleted Successfully');
                })
                .catch(err => console.log(err));
       }
    },

    addArticle() {
        if (this.edit === false) {
            axios.post('/api/articles', {
                title: this.article.title,
                body: this.article.body
            })
            .then(res => {
                this.fetchArticles();
                alert('Article Added Successfully');
            })
            .catch(err => console.log(err));
        } else {
            axios.put(`/api/articles/${this.article_id}`, {
                id: this.article.id,
                title: this.article.title,
                body: this.article.body
            })
            .then(res => {
                this.fetchArticles();
                alert('Article Updated Successfully');
            })
            .catch(err => console.log(err));
        }

        this.article.title = '';
        this.article.body = '';
    },

    editArticle(article) {
        this.edit = true;
        this.article.id = article.id;
        this.article_id = article.id
        this.article.title = article.title;
        this.article.body = article.body;
    }
  }
};
</script>
