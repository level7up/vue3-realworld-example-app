<template>
    <div>
        <div
            class="container"
        >
        <!-- article.revisions TABLE -->
        <div class="revisions-wrapper">
            <table class="revisions-table">
                <thead>
                    <tr>
                        <th class="revisions-th">Revision #</th>
                        <th class="revisions-th">Title</th>
                        <th class="revisions-th">Description</th>
                        <th class="revisions-th">Body</th>
                        <th class="revisions-th">User ID</th>
                        <th class="revisions-th">Created At</th>
                        <th class="revisions-th">Action</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="rev in article.revisions" :key="rev.id">
                        <td class="revisions-td">{{ rev.revision_number }}</td>
                        <td class="revisions-td">{{ rev.title }}</td>
                        <td class="revisions-td">{{ rev.description }}</td>
                        <td class="revisions-td">{{ rev.body }}</td>
                        <td class="revisions-td">{{ rev.author.username }}</td>
                        <td class="revisions-td">{{ new Date(rev.createdAt).toLocaleString() }}</td>
                        <td class="revisions-td">
                            <form @submit.prevent="onSubmit(rev)">
                                <button type="submit" class="btn btn-primary btn-sm"> Revert</button>
                            </form>
                        </td>

                    </tr>
                    <tr v-if="!article.revisions.revisions?.length">
                        <td class="no-revisions" colspan="6">No revisions found</td>
                    </tr>
                </tbody>
            </table>
        </div>
        </div>

    </div>

</template>

<script setup lang="ts">
import Swal from "sweetalert2";
import { computed, onMounted, reactive } from "vue";
import { useRoute, useRouter } from 'vue-router'
import { api } from "src/services";

const router = useRouter()

interface ArticleRevision {
    id: number;
    article_id: number;
    user_id: number;
    title: string;
    slug: string;
    description: string;
    body: string;
    revision_number: number;
    created_at: string;
    updated_at: string;
}

interface ArticleState {
    title: string;
    body: string;
    revisions: ArticleRevision[];
}

const route = useRoute();
const slug = computed<string>(() => route.params.slug as string);

const article: ArticleState = reactive({
    title: "",
    body: "",
    revisions: [],
});

async function fetchArticleRevisions(slug: string) {
    const tt: any = await api.articles
        .getArticleRevisions(slug)
        .then((res) => res.data.revisions);
    article.revisions = (tt) as ArticleRevision[];
}

async function onSubmit(rev) {
    let res = await api.articles.revertArticle(slug.value, rev.id).then(res => res.data.article)
    Swal.fire({
        title: "Good job!",
        text: "Vision Has been set successfully",
        icon: "success"
    });
    return router.push({ name: 'article', params: { slug: res.slug } })
}
onMounted(async () => {
    if (slug.value) await fetchArticleRevisions(slug.value);
});
</script>

<style scoped>
.revisions-wrapper {
    margin: 24px auto;
    padding: 24px 0;
    max-width: 960px;
}

.revisions-table {
    width: 100%;
    border-collapse: collapse;
}

.revisions-th {
    text-align: left;
    padding: 8px;
    border-bottom: 1px solid #e5e7eb;
    font-weight: 600;
    font-size: 14px;
    color: #374151;
}

.revisions-td {
    padding: 8px;
    border-bottom: 1px solid #f0f2f5;
    vertical-align: top;
    font-size: 14px;
    color: #111827;
}

.no-revisions {
    padding: 16px;
    text-align: center;
    color: #6b7280;
}
</style>
