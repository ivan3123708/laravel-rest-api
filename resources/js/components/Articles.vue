<template>
    <div>
        <h1>Articles</h1>
        <form @submit.prevent="addArticle" class="mb-3">
            <div class="form-group">
                <input type="text" class="form-control" placeholder="Title" v-model="article.title">
            </div>
            <div class="form-group">
                <textarea type="text" class="form-control" placeholder="Body" v-model="article.body"></textarea>
            </div>
            <button type="submit" class="btn btn-success btn-block">Save</button>
        </form>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li :class="{ disabled: !pagination.prev_page_url }" class="page-item">
                    <a @click="fetchArticles(pagination.prev_page_url)" class="page-link" href="#" aria-label="Previous">
                        <span aria-hidden="true">&laquo;</span>
                        <span class="sr-only">Previous</span>
                    </a>
                </li>
                <li class="page-item">
                    <a class="page-link disabled">Page {{ pagination.current_page }} of {{ pagination.last_page }}</a>
                </li>
                <li :class="{ disabled: !pagination.next_page_url }" class="page-item">
                    <a @click="fetchArticles(pagination.next_page_url)" class="page-link" href="#" aria-label="Next">
                        <span aria-hidden="true">&raquo;</span>
                        <span class="sr-only">Next</span>
                    </a>
                </li>
            </ul>
        </nav>
        <div class="card card-body mb-2" v-for="(article, index) in articles" :key="index">
            <h3>{{ article.title }}</h3>
            <p>{{ article.body }}</p>
            <hr>
            <button class="btn btn-warning mb-2" @click="editArticle(article)">Edit</button>
            <button class="btn btn-danger" @click="deleteArticle(article.id)">Delete</button>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                articles: [],
                article: {
                    id: '',
                    title: '',
                    body: ''
                },
                article_id: '',
                pagination: {},
                edit: false
            }
        },
        methods: {
            fetchArticles(page_url) {
                let vm = this;
                page_url = page_url || '/api/articles';

                fetch(page_url)
                    .then((res) => res.json())
                    .then((res) => {
                        this.articles = res.data;
                        vm.makePagination(res.meta, res.links);
                    })
                    .catch((err) => console.log(err));
            },
            makePagination(meta, links) {
                let pagination = {
                    current_page: meta.current_page,
                    last_page: meta.last_page,
                    next_page_url: links.next,
                    prev_page_url: links.prev
                }

                this.pagination = pagination;
            },
            addArticle() {
                if (this.edit) {
                    fetch('/api/articles', {
                        method: 'PUT',
                        body: JSON.stringify(this.article),
                        headers: {
                            'Content-Type': 'application/json'
                        }
                    })
                    .then((res) => res.json())
                    .then((data) => {
                        alert('Article edited');
                        this.article.title = '';
                        this.article.body = '';
                        this.fetchArticles();
                    })
                    .catch((err) => console.log(err));
                } else {
                    fetch('/api/articles', {
                        method: 'POST',
                        body: JSON.stringify(this.article),
                        headers: {
                            'Content-Type': 'application/json'
                        }
                    })
                    .then((res) => res.json())
                    .then((data) => {
                        alert('Article added');
                        this.article.title = '';
                        this.article.body = '';
                        this.fetchArticles();
                    })
                    .catch((err) => console.log(err));
                }
            },
            editArticle(article) {
                this.edit = true;
                this.article.article_id = article.id;
                this.article.id = article.id;
                this.article.title = article.title;
                this.article.body = article.body;
            },
            deleteArticle(id) {
                fetch(`/api/articles/${id}`, { method: 'DELETE' })
                    .then((res) => res.json())
                    .then((data) => {
                        alert('Article deleted');
                        this.fetchArticles();
                    })
                    .catch((err) => console.log(err));
            }
        },
        created() {
            this.fetchArticles();
        }
    }
</script>
