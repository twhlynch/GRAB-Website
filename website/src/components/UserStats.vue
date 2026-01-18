<script>
import { getUserInfoRequest } from '../requests/GetUserInfoRequest';
import { getUserStatsRequest } from '../requests/GetUserStatsRequest';

export default {
	props: {
		user_id: String,
	},
	data() {
		return {
			stats: {},
			info: {},
			loaded: false,
		};
	},
	computed: {
		is_creator() {
			return this.info.is_creator;
		},
		is_moderator() {
			return this.info.is_moderator;
		},
		is_developer() {
			return this.info.is_developer || this.info.is_admin;
		},
		user_name() {
			return this.info.user_name;
		},
		finishes_tiers() {
			return this.stats.levels_finished_by_difficulty_tier;
		},
		top_1_tiers() {
			return this.stats.top1_by_difficulty_tier;
		},
		top_10_tiers() {
			return this.stats.top10_by_difficulty_tier;
		},
		raw_tiers() {
			return this.stats.rating.per_tier_raw;
		},
		weighted_tiers() {
			return this.stats.rating.per_tier_weighted;
		},
	},
	methods: {
		async get_details() {
			this.loaded = false;
			this.stats = {};
			this.info = {};

			const stats_promise = getUserStatsRequest(this.$api_server_url, this.user_id);
			const info_promise = getUserInfoRequest(this.$api_server_url, this.user_id);
			const [stats, info] = await Promise.all([stats_promise, info_promise]);

			if (!stats) window.toast('Failed to load user stats', 'error');
			if (!info) window.toast('Failed to load user info', 'error');
			if (!stats || !info) return;

			this.stats = stats;
			this.info = info;
			this.loaded = true;
		},
	},
	created() {
		this.get_details();
	},
};
</script>

<template>
	<div class="stats-container" v-if="loaded">
		<div class="user-info">
			<div class="user-name">
				<span>{{ user_name }}</span>
				<img v-if="is_creator" title="Creator" alt="creator" class="creator-icon" src="./../assets/icons/checkmark.svg" />
				<span v-if="is_moderator" title="Moderator" class="moderator-icon">M</span>
				<span v-if="is_developer" title="Developer" class="developer-icon">D</span>
				<a class="profile-button" :href="'levels?tab=tab_other_user&user_id=' + user_id">Levels</a>
			</div>
		</div>

		<div class="finishes">
			<h2>Finishes</h2>
			<span>{{ stats.levels_finished_unique }} / {{ stats.levels_played_unique }} Levels Finished</span>
			<span>First finishes: {{ stats.first_finisher_count }}</span>
			<div class="by-tier">
				<div class="easy">{{ finishes_tiers.easy }}</div>
				<div class="medium">{{ finishes_tiers.medium }}</div>
				<div class="hard">{{ finishes_tiers.hard }}</div>
				<div class="veryhard">{{ finishes_tiers.veryhard }}</div>
				<div class="impossible">{{ finishes_tiers.impossible }}</div>
			</div>
		</div>

		<div class="records">
			<h2>Records</h2>
			<div>
				<div>
					<span>{{ stats.top1_total }} World Records</span>
					<div class="by-tier">
						<div class="easy">{{ top_1_tiers.easy }}</div>
						<div class="medium">{{ top_1_tiers.medium }}</div>
						<div class="hard">{{ top_1_tiers.hard }}</div>
						<div class="veryhard">{{ top_1_tiers.veryhard }}</div>
						<div class="impossible">{{ top_1_tiers.impossible }}</div>
					</div>
				</div>
				<div>
					<span>{{ stats.top10_total }} Top 10s</span>
					<div class="by-tier">
						<div class="easy">{{ top_10_tiers.easy }}</div>
						<div class="medium">{{ top_10_tiers.medium }}</div>
						<div class="hard">{{ top_10_tiers.hard }}</div>
						<div class="veryhard">{{ top_10_tiers.veryhard }}</div>
						<div class="impossible">{{ top_10_tiers.impossible }}</div>
					</div>
				</div>
			</div>
		</div>

		<div class="activity">
			<h2>Activity</h2>
			<span>{{ stats.levels_finished_unique }} Levels Finished</span>
			<span>Active: {{ stats.last_active_timestamp }}</span>
			<span>Streak: {{ stats.streak_days_current }}</span>
			<span>Longest Streak: {{ stats.streak_days_longest }}</span>
		</div>

		<div class="levels">
			<h2>Levels</h2>
			<span>Plays: {{ stats.creator_total_plays_received }}</span>
			<span>Unique plays: {{ stats.creator_total_plays_received_unique }}</span>
			<span>Finishes: {{ stats.creator_total_finishes_received }}</span>
			<span>Unique finishes: {{ stats.creator_total_finishes_received_unique }}</span>
			<span>Likes: {{ stats.creator_total_likes_received }}</span>
			<span>Tips: {{ stats.creator_total_tips_received_count }}</span>

			<span>Own levels published: {{ stats.publish_verification_total_count }}</span>
			<span>Others levels published: {{ stats.publish_verification_non_own_count }}</span>
			<span>Levels rated: {{ stats.ratings_given_unique }}</span>
			<span>Likes given: {{ stats.likes_given_unique }}</span>
			<span>Tips given: {{ stats.tips_given_unique }}</span>
		</div>

		<div class="score">
			<h2>Score</h2>
			<div>
				<div>
					<span>Raw</span>
					<div class="by-tier">
						<div class="easy">{{ Math.floor(raw_tiers.easy) }}</div>
						<div class="medium">{{ Math.floor(raw_tiers.medium) }}</div>
						<div class="hard">{{ Math.floor(raw_tiers.hard) }}</div>
						<div class="veryhard">{{ Math.floor(raw_tiers.veryhard) }}</div>
						<div class="impossible">{{ Math.floor(raw_tiers.impossible) }}</div>
					</div>
				</div>
				<div>
					<span>Weighted</span>
					<div class="by-tier">
						<div class="easy">{{ Math.floor(weighted_tiers.easy) }}</div>
						<div class="medium">{{ Math.floor(weighted_tiers.medium) }}</div>
						<div class="hard">{{ Math.floor(weighted_tiers.hard) }}</div>
						<div class="veryhard">{{ Math.floor(weighted_tiers.veryhard) }}</div>
						<div class="impossible">{{ Math.floor(weighted_tiers.impossible) }}</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style scoped>
/* layout */
.stats-container {
	overflow-wrap: break-word;
	width: 100%;
	height: 100%;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	gap: 1em;
}
.stats-container > div {
	width: 100%;
	height: fit-content;
	display: flex;
	justify-content: space-between;
	flex-direction: column;
	align-items: center;
	background-color: var(--hover);
	border-radius: 15px;
	padding: 0.5rem 1rem;
}
.user-name {
	width: 100%;
	font-size: 20px;
	font-style: bold;
	display: flex;
	flex-direction: row;
	align-items: center;
	justify-content: flex-start;
	gap: 5px;
}
.by-tier {
	display: flex;
	flex-direction: row;
	gap: 0.5em;
	> div {
		background-color: var(--hover);
		width: 3rem;
		height: 3rem;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 15px;
	}
	.impossible {
		background-color: #7f007fa0;
	}
	.veryhard {
		background-color: #ea0000a0;
	}
	.hard {
		background-color: #f19400a0;
	}
	.medium {
		background-color: #e1c800a0;
	}
	.easy {
		background-color: #2bba84a0;
	}
}
.records,
.score {
	> div {
		width: 100%;
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: space-around;
		gap: 5px;
		flex-wrap: wrap;
	}
}
/* icons */
.creator-icon {
	width: 20px;
	height: 20px;
}
.moderator-icon {
	width: 20px;
	height: 20px;
	line-height: 20px;
	background-color: #de9343;
	border-radius: 50%;
	font-size: 14px;
	font-weight: 600;
	display: flex;
	align-items: center;
	justify-content: center;
}
.developer-icon {
	width: 20px;
	height: 20px;
	line-height: 20px;
	background-color: #dd3619;
	border-radius: 50%;
	font-size: 14px;
	font-weight: 600;
	display: flex;
	align-items: center;
	justify-content: center;
}
.profile-button {
	padding: 2px 8px 1px 8px;
	font-weight: bold;
	background-color: var(--blue);
	color: white;
	font-size: 12px;
	border-radius: 15px;
	cursor: pointer;
}
</style>
