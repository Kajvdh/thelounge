<template>
	<div class="mentions-popup">
		<h3 class="mentions-popup-title">Recent mentions</h3>
		<template v-if="resolvedMessages.length === 0">
			<p>There are no recent mentions.</p>
		</template>
		<template v-for="message in resolvedMessages" v-else>
			<div :key="message.id" :class="['msg', message.type]">
				<span class="from">
					<Username :user="message.from" />
					<template v-if="message.channel">
						in {{ message.channel.channel.name }} on {{ message.channel.network.name }}
					</template>
					<template v-else>
						in unknown channel
					</template>
				</span>
				<span :aria-label="message.time | localetime" class="time tooltipped tooltipped-n">
					{{ messageTime(message.time) }}
				</span>
				<button @click="hideMention(message)">Hide</button>
				<div class="content" dir="auto">
					<ParsedMessage :network="null" :message="message" />
				</div>
			</div>
		</template>
	</div>
</template>

<script>
import Username from "./Username.vue";
import ParsedMessage from "./ParsedMessage.vue";
import socket from "../js/socket";
import dayjs from "dayjs";
import relativeTime from "dayjs/plugin/relativeTime";

dayjs.extend(relativeTime);

export default {
	name: "Mentions",
	components: {
		Username,
		ParsedMessage,
	},
	computed: {
		resolvedMessages() {
			const messages = this.$store.state.mentions.slice().reverse();

			for (const message of messages) {
				message.channel = this.$store.getters.findChannel(message.chanId);
			}

			return messages;
		},
	},
	mounted() {
		socket.emit("mentions:get");
	},
	methods: {
		messageTime(time) {
			return dayjs(time).fromNow();
		},
		hideMention(message) {
			this.$store.state.mentions.splice(
				this.$store.state.mentions.findIndex((m) => m.msgId === message.msgId),
				1
			);

			socket.emit("mentions:hide", message.msgId);
		},
	},
};
</script>
