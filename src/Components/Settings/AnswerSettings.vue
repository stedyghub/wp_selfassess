<template>
	<div class="answers elements">
		<h3 class="toggle-contents" @click="toggleContents">
			{{ text.answers }}
		</h3>

		<div
			ref="contentsCounterpart"
			class="contentsCounterpart"
			@click="toggleContents">
			<i class="icon icon-edit" />
			{{ text.edit_answers }}
		</div>

		<div ref="contents" class="contents hidden">
			<draggable
				v-model="answerList"
				handle=".drag-handle"
				draggable=".answer"
				@input="emitAnswerListUpdate">
				<div
					v-for="(answer, i) in answerList"
					:key="i"
					class="answer element">
					<table>
						<thead>
							<tr>
								<th colspan="2">
									<i class="icon icon-align-justify drag-handle" />
									<p class="open-panel">
										{{ answer.data.content }}
									</p>
								</th>
							</tr>
						</thead>
						<tbody>
							<tr>
								<td>
									<label :for="'answerContent_' + answer.id">
										{{ text.answer_content }}
									</label>
								</td>
								<td>
									<input
										:id="'answerContent_' + answer.id"
										v-model="answer.data.content"
										type="text"
										:placeholder="text.answer_content"
										@input="emitAnswerListUpdate">
								</td>
							</tr>
							<tr>
								<td>
									<label :for="'answerValue_' + answer.id">
										{{ text.answer_value }}
									</label>
								</td>
								<td>
									<input
										:id="'answerValue_' + answer.id"
										v-model="answer.data.value"
										type="number"
										:placeholder="text.answer_value"
										@input="emitAnswerListUpdate">
								</td>
							</tr>
							<tr>
								<td colspan="2">
									<button class="button button-danger" @click="deleteAnswer(answer.id)">
										{{ text.delete }}
									</button>
								</td>
							</tr>
						</tbody>
					</table>
				</div>

				<div slot="footer">
					<button
						:class="{ loading: addingAnswerInProgress }"
						:disabled="addingAnswerInProgress"
						class="button button-primary"
						@click="addAnswer">
						{{ text.add_answer }}
					</button>
				</div>
			</draggable>
		</div>
	</div>
</template>

<script>
/* global ajaxurl */
/* global jQuery */
/* global Vue */
/* global gifttest */
__webpack_public_path__ = gifttest.vue_components_path // eslint-disable-line

const Vuedraggable = () => import(/* webpackChunkName: "Vuedraggable" *//* webpackPrefetch: true */'vuedraggable')
const Utilities = () => import(/* webpackChunkName: "Utilities" */'../Utilities.js')

export default {
	name: 'AnswerSettings',
	components: {
		draggable: Vuedraggable,
	},
	props: {
		value: {
			type: Array,
			required: true,
			default: function() { return [] },
		},
		questionaireId: {
			type: Number,
			required: true,
		},
		text: {
			type: Object,
			required: true,
		},
	},
	data: function() {
		return {
			answerList: this.value,
			addingAnswerInProgress: false,
		}
	},
	watch: {
		value: function(newValue, oldValue) {
			this.answerList = newValue
		},
	},
	beforeMount: function() {
		const self = this
		// load requried modules
		Vuedraggable()
		// load displayMessage function
		Utilities().then(utilities => {
			self.displayMessage = utilities.default.displayMessage
		})
	},
	methods: {
		/**
		 * This is a dummy function that will be replace asynchronous
		 */
		displayMessage() {},
		deleteAnswer(answerId) {
			const self = this

			jQuery.each(self.answerList, function(i, answer) {
				if (answer.id === answerId) {
					Vue.delete(self.answerList, i)
					return false
				}
			})
		},
		highestAnswerId() {
			let highestId = 0
			jQuery.each(this.answerList, function(i, answer) {
				if (answer.id > highestId) highestId = answer.id
			})
			return highestId
		},
		addAnswer() {
			const self = this

			// only one request allowed at a time
			if (self.addingAnswerInProgress) return
			else self.addingAnswerInProgress = true

			/* create new answer */
			// collect data
			const requestData = {
				_ajax_nonce: gifttest._ajax_nonce.create_answer,
				action: 'gifttest_create_questionaire_answer',
				id: self.highestAnswerId() + 1,
				questionaire_id: self.questionaireId,
			}

			// do request
			jQuery.post(ajaxurl, requestData, function(response) {
				if (response.status === 'success') {
					self.answerList.push(response.data)
				} else {
					self.displayMessage(response.message, response.status)
				}
				// loading done
				self.addingAnswerInProgress = false
			}, 'json')
		},
		toggleContents() {
			jQuery(this.$refs.contents).toggle()
			jQuery(this.$refs.contentsCounterpart).toggle()
		},
		emitAnswerListUpdate() {
			this.$emit('input', this.answerList)
		},
	},
}
</script>
