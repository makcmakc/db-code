/*
* Допустим, есть задача: сделать форму оформления заказа для интернет-магазина. 
* Далее представлен фрагмент кода(а точнее компонент), который выполняет валидацию 
* полей ввода этой формы, а также отображает тип платежной системы.
*
* Конечно, это не конечный продукт, и тут есть над чем еще работать.
* Ссылка на HeadHunter профиль - 
* https://yoshkar-ola.hh.ru/resume/f97e7518ff06dd0ce80039ed1f33584364486e
* Ссылка на гитхаб проекта - https://github.com/makcmakc/code-example
* Полный прототип можно увидеть здесь - https://code-example-1c893.firebaseapp.com
*/


<template>
	<div>

		<form 
			class="form vld-parent" 
			v-on:submit.prevent="onSubmit">

			<div class="page-title">Payment</div>

			<div class="payment-notice">
				<svg><use xlink:href="#payment-notice-icon"></use></svg>
				<span>This is a secure 128-bit SSL encrypted payment</span>
			</div>
			
			<div class="payment-wrap">
				<h5>Cardholder Name</h5>
					<input 
						placeholder="Name as it appears on your card" 
						type="text" 
						class="form-control"
						v-model.trim="$v.firstname.$model"
						:class="{'is-invalid':$v.firstname.$error, 'is-valid':!$v.firstname.$invalid }"/>

						<div class="valid-feedback">Your name is vaild!</div>

						<div class="invalid-feedback">
							<span v-if="!$v.firstname.required">First name is required.</span>
							<span v-if="!$v.firstname.minLength">First name must have at least {{ $v.firstname.$params.minLength.min}} letters.</span>		
							<span v-if="!$v.firstname.maxLength">First name must have at most {{ $v.firstname.$params.maxLength.max}} letters.</span>		
						</div>						

				<h5>Card Number</h5>
				<div class="form-group">
					<input 
						type="text" 
						class="form-control" 
						placeholder="XXXX XXXX XXXX XXXX"
						v-mask="generateCardNumberMask"
						id="cardNumber" 								
						v-model.trim="$v.cardNumber.$model"
						:class="{'is-invalid':$v.cardNumber.$error, 'is-valid':!$v.cardNumber.$invalid }">

						<div class="valid-feedback">Card is vaild!</div> 

						<div class="invalid-feedback">
							<span v-if="!$v.cardNumber.required">Card Number is required.</span>
							<span v-if="!$v.cardNumber.minLength">Card Number must have at least {{ $v.cardNumber.$params.minLength.min}} numbers.</span>
						</div>

						<svg class="payment-method-icon">
							<use 
								v-bind:xlink:href="'#' + getCardType"
								v-if="getCardType" 
								v-bind:key="getCardType">
							</use>
						</svg>
				</div>

				<div>
					<h5>Expire Date</h5>
					<div class="form-group">
						<input 
							class="form-control expire-date" 
							v-mask="expireDateMask"
							placeholder="MM / YY"
							v-model.trim="$v.expireDate.$model"
							:class="{'is-invalid':$v.expireDate.$error, 'is-valid':!$v.expireDate.$invalid }"/>

						<div class="valid-feedback">Expire Date is vaild!</div> 

						<div class="invalid-feedback">
						<!-- /* Read the comments in the "script" section */
							<span v-if="!$v.expireDate.cardMonth">Card Month is not valid.</span>
							<span v-if="!$v.expireDate.cardYear">Card Yaer is not valid.</span>
						-->
							<span v-if="!$v.expireDate.cardDateExpire">Expire Date is not valid.</span>
							<span v-if="!$v.expireDate.required">Expire Date is required.</span>
						</div>

					</div>
				</div>


				<div class="security-code" >
					<h5>Security Code</h5>
					<div class="form-group">
						<input 
							class="form-control zip" 
							placeholder="CCV"
							v-mask="ccvMask"
							v-model.trim="$v.cardCvv.$model"
							:class="{'is-invalid':$v.cardCvv.$error, 'is-valid':!$v.cardCvv.$invalid }" />

						<div class="valid-feedback">CVV is vaild!</div> 

						<div class="invalid-feedback">
							<span v-if="!$v.cardCvv.required">Security Code is required.</span>
							<span v-if="!$v.cardCvv.minLength">CVV must have at least {{ $v.cardCvv.$params.minLength.min}} letters.</span>
						</div>

					</div>		
				</div>

				<div class="btn-wrap">
					<button type="submit" class="btn">Pay Securely</button>
				</div>				
			</div>

		</form>

		<Icons />
		
	</div>
</template>


<script>
	import { required, minLength, maxLength } from 'vuelidate/lib/validators'
	import { mask } from 'vue-the-mask'	
	import Icons from '../components/svgIcons.vue'

	export default {
		name: 'payment',
		data: () =>  ({

			// Genarall variables for component
			firstname: '',
			cardNumber: "",
			cardCvv: "",
			expireDate: "",
			minCardYear: new Date().getFullYear(),

			// Masks
			amexCardMask: "#### ###### #####",
			otherCardMask: "#### #### #### ####",
			expireDateMask: "## / ##",
			ccvMask: "####",

			// For loader
			fullPage: false
		}),

		// Validation conditions
		validations: {
			firstname: {
				required,
				minLength: minLength(5),
				maxLength: maxLength(30)
			},
			cardNumber: {
				required,
				minLength: minLength(15)
			},
			expireDate: {
				required,

				// Here can be two ways

				// First one: to do two different methods, and then a user will be able to understand that exactly went wrong 
				/*
				cardYear(value) {
					if (value === '') return true
					if (this.expireDate.slice(5, 7) < String(this.minCardYear).slice(0, 2)) {
						return false
					}
					return true
					
				},
				cardMonth(value) {
					if (value === '') return true
					if (this.expireDate.slice(0, 2) <= 0 || this.expireDate.slice(0, 2) > 12) {
						return false
					}
					return true
				},
				*/

				// Second one: to do one general method, I find this one more attractive
				cardDateExpire(value) {
					if (value === '') return true
					// Check that the year can't be le than current or the month can't be less than 01 or more than 12
					if (this.expireDate.slice(5, 7) < String(this.minCardYear).slice(0, 2) ||
						(this.expireDate.slice(0, 2) <= 0 || this.expireDate.slice(0, 2) > 12)) {
						return false
					}
					return true						
				}
			},
			cardCvv: {
				required,
				minLength: minLength(3)
			}						
		},	
		directives: { mask },
		components: { Icons },
		computed: {
			// Getting the card type by going through regular expression
			getCardType () {
				let number = this.cardNumber 
				let re = new RegExp("^4") 
				if (number.match(re) != null) return "visa" 

				re = new RegExp("^(34|37)") 
				if (number.match(re) != null) return "amex" 

				re = new RegExp("^5[1-5]") 
				if (number.match(re) != null) return "mastercard" 

				re = new RegExp("^6011") 
				if (number.match(re) != null) return "discover" 

				return ''  // default type
			},
			// According to with the type of card select a mask
			generateCardNumberMask () {
				return this.getCardType === "amex" ? this.amexCardMask : this.otherCardMask 
			}
		},	
		methods: {

			// Data processing emulation by "Back-End"
			onSubmit() {
				// Set Loader Props
				let loader = this.$loading.show({
					container: this.fullPage ? null : this.$refs.formContainer,
					canCancel: false,
					onCancel: this.onCancel,
					loader: 'dots',
					color: '#7A3FA8',
					width: 128,
					height: 128,
					backgroundColor: '#aaa',
					opacity: 0.5,
					zIndex: 999
				});

				// Simulate AJAX
				setTimeout(() => {
					this.$router.push('/success')
					loader.hide()
				}, 4000)   		
			}
		}
	}
</script>


<style scoped>
/* style patch */
.btn-wrap {
	margin-top: 80px 
}
.zip {
	margin-top: 20px;
}
</style>

