<template>
	<div class="email-container">
		<h1>User Emails</h1>
		<div v-if="emails.length === 0">
			<p>No emails found for this secret.</p>
		</div>
		<div v-else>
			<div v-for="email in emails" :key="email.emailId" class="email-card">
				<div class="email-header">
					<h2>主题: {{ email.subject }}</h2>
					<p>发件人: {{ email.sendEmail }}</p>
					<p>收件人: {{ email.toEmail }}</p>
					<p>收件时间: {{ email.createTime }}</p>
				</div>
				<div class="email-body">
					<h3>邮件内容:</h3>
					<div class="email-content" v-html="email.content"></div>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import http from '../../axios/index'; // Assuming request.js handles API calls

const emails = ref([]);
const route = useRoute();

onMounted(async () => {
	const secret = route.query.secret;
	const limit = route.query.limit;
	if (secret) {
		try {
			const res = await http.get(`/public/showemail`, { params: { secret, limit } });
			if (res.code === 200) {
				emails.value = res.data.map(email => ({
					...email,
					content: extractBodyContent(email.content)
				}));
			} else {
				console.error('Error fetching emails:', res.message);
			}
		} catch (error) {
			console.error('API call failed:', error);
		}
	} else {
		console.error('Secret is missing from the URL.');
	}
});

// Helper function to extract body content from HTML
function extractBodyContent(htmlString) {
	const doc = new DOMParser().parseFromString(htmlString, 'text/html');
	return doc.body.innerHTML;
}
</script>

<style scoped>
.email-container {
	max-width: 800px;
	margin: 20px auto;
	padding: 20px;
	background-color: #f9f9f9;
	border-radius: 8px;
	box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

h1 {
	text-align: center;
	color: #333;
	margin-bottom: 30px;
}

.email-card {
	background-color: #fff;
	border: 1px solid #ddd;
	border-radius: 8px;
	margin-bottom: 20px;
	overflow: hidden;
	box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
}

.email-header {
	background-color: #eef;
	padding: 15px;
	border-bottom: 1px solid #ddd;
}

.email-header h2 {
	margin-top: 0;
	color: #0056b3;
}

.email-header p {
	margin: 5px 0;
	color: #555;
	font-size: 0.9em;
}

.email-body {
	padding: 15px;
}

.email-body h3 {
	margin-top: 0;
	margin-bottom: 10px;
	color: #333;
}

.email-content {
	padding: 15px;
	line-height: 1.6;
	color: #333;
	word-wrap: break-word; /* Ensure content wraps */
}

.email-content >>> img {
	max-width: 100%;
	height: auto;
}

.email-content >>> table {
	width: 100% !important;
	border-collapse: collapse;
}

.email-content >>> th,
.email-content >>> td {
	border: 1px solid #ccc;
	padding: 8px;
	text-align: left;
}
</style>
