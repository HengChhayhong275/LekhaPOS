<script>

    import SideMenu from '../components/MenuSideBar.vue';
    import { Icon } from '@iconify/vue';
    import { Modal } from 'bootstrap'
    import axios from 'axios';

    export default {

        components: {
            SideMenu,
            Icon
        },
        data(){
            return{
                menu: '',
                order: '',
                orderHistory: '',
                orderNo: '#0000',
                tableNo: '0',
                modal: '',
                modalData: '',
                modalQuantity: '',
                modalSelectedSize: '',
                modalSelectedAddon: [],
                mobileSidebarToggled: false
            }
        },
        mounted() {
		
		axios.get('/asset/data/pos/customer-order.json').then((response) => {
			this.menu = response.data;
			this.order = response.data.order;
			this.orderNo = response.data.orderNo;
			this.orderHistory = response.data.orderHistory;
			this.tableNo = response.data.tableNo;
		});
	},
        methods: {
            toggleMobileSidebar: function(){
                this.mobileSidebarToggled = !this.mobileSidebarToggled;
            },
            getOrderTotal: function(){
                return (this.order) ? this.order.length : 0;
            },
            getOrderHistoryTotal : function (){
                return (this.orderHistory) ? this.orderHistory.length : 0;
            },
            getSubTotalPrice: function (){
                var value = 0;
                for (var i=0; i < this.order.length; i++){
                    value += parseFloat(this.order[i].price) * parseInt(this.order[i].quantity);
                }
                return value.toFixed(2);
            },
            getTaxesPrice: function() {
                var value = 0;
                    for (var i = 0; i < this.order.length; i++) {
                        value += parseFloat(this.order[i].price) * parseInt(this.order[i].quantity) * .06;
                    }
                return value.toFixed(2);
		    },
            getTotalPrice: function() {
                var value = 0;
                    for (var i = 0; i < this.order.length; i++) {
                        value += parseFloat(this.order[i].price) * parseInt(this.order[i].quantity);
                        value += parseFloat(this.order[i].price) * parseInt(this.order[i].quantity) * .06;
                    }
                return value.toFixed(2);
            },
            deductQty: function(event, id) {
                event.preventDefault();
                for (var i = 0; i < this.order.length; i++) {
                    if (this.order[i].id == id) {
                        var newQty = parseInt(this.order[i].quantity) - 1;
                        
                        if (newQty < 1) {
                            newQty = 1;
                        }
                        this.order[i].quantity = newQty;
                    }
                }
            },
            addQty: function(event, id) {
                event.preventDefault();
                
                for (var i = 0; i < this.order.length; i++) {
                    if (this.order[i].id == id) {
                        var newQty = parseInt(this.order[i].quantity) + 1;
                        
                        this.order[i].quantity = newQty;
                    }
                }
            },
            showType: function(event, type) {
                event.preventDefault();
                
                for (var i = 0; i < this.menu.category.length; i++) {
                    if (this.menu.category[i].type == type) {
                        this.menu.category[i].active = true;
                    } else {
                        this.menu.category[i].active = false;
                    }
                }
                for (var i = 0; i < this.menu.food.length; i++) {
                    if (this.menu.food[i].type == type || type == 'all') {
                        this.menu.food[i].hide = false;
                    } else {
                        this.menu.food[i].hide = true;
                    }
                }
            },
            showFoodModal: function(event, id) {
                event.preventDefault();
                
                for (var i = 0; i < this.menu.food.length; i++) {
                    if (this.menu.food[i].id == id) {
                        this.modalData = this.menu.food[i];
                    }
                }
                if (this.modalData.options && this.modalData.options.size) {
                    this.modalSelectedSize = this.modalData.options.size[0].text;
                }
                this.modalQuantity = 1;
                this.modalSelectedAddon = [];
                this.modal = new Modal(this.$refs.modalPosItem);
                this.modal.show();
            },
            addModalQty: function(event) {
                event.preventDefault();
                
                this.modalQuantity = this.modalQuantity + 1;
            },
            deductModalQty: function(event) {
                event.preventDefault();
                
                var newQty = parseInt(this.modalQuantity) - 1;
            
                if (newQty < 1) {
                    newQty = 1;
                }
                this.modalQuantity = newQty;
            },
            addToCart: function(event) {
                event.preventDefault();
                
                this.modal.hide();
                
                var options = [];
                var extraPrice = 0;
                if (this.modalSelectedSize) {
                    var option = {
                        "key": "size",
                        "value": this.modalSelectedSize
                    };
                    options.push(option);
                }
                if (this.modalSelectedAddon) {
                    for (var i = 0; i < this.modalSelectedAddon.length; i++) {
                        var option = {
                            "key": "addon",
                            "value": this.modalSelectedAddon[i]
                        };
                        options.push(option);
                    }
                }
                
                this.order.push({
                    "id": (this.order.length + 1),
                    "image": this.modalData.image,
                    "title": this.modalData.title,
                    "price": this.modalData.price,
                    "quantity": this.modalQuantity,
                    "options": options
                });
                
                setTimeout(() => {
                    this.$refs.posSidebarBody.$el.scrollTop = 9999;
                    this.$refs.posSidebarBody.ps.update();
                }, 500);
            },
            toggleConfirmation: function(event, id, value) {
                event.preventDefault();
                
                for (var i = 0; i < this.order.length; i++) {
                    if (this.order[i].id == id) {
                        this.order[i].confirmation = value;
                    }
                }
            },
		    removeOrder: function(event, id) {
                event.preventDefault();
                
                for (var i = 0; i < this.order.length; i++) {
                    if (this.order[i].id == id) {
                        this.order.splice(i, 1);
                    }
                }
            }
        }
    }

</script>
<template>
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
        <div class="bg-white flex">
            <div class="w-1/6">
                <SideMenu/>
            </div>
            <div class="w-7/12">

            </div>
            <!-- BEGIN CHECKOUT SIDE  -->
            <div class="w-1/4 fixed top-0 right-0">
                <div class="bg-inherite w-full h-screen flex flex-col">
                    
                    <div class="absolute header-checkout bg-slate-300  mx-2 mt-5 rounded-lg">
                        <!-- header  -->
                       <div class="header-check max-h-12 w-full rounded-t-lg  flex justify-between">
                        <!-- ORDER TABLE  -->
                            <div class="flex justify-center items-center ms-2">
                                <Icon class="" icon="fa6-solid:plate-wheat" color="#9ca3af" />
                                <p class="text-white ms-1 my-2">Table1</p>
                            </div>
                            <!-- ORDER NUMBER  -->
                            <div class="order-num text-white mx-4 my-2 px-3 rounded">
                                <h1>Order : <span class="font-semibold">#8888</span></h1>
                            </div>
                       </div>
                       <!-- body  -->
                       <div class="bg-white h-4/6">
                            <div class="bg-white h-10">
                                <h1 class="text-black h-full text-center flex justify-center items-center">New Order ({{getOrderTotal()}})</h1>
                            </div>
                            <!-- begin pos table  -->
                            <div class="py-0 px-8 " v-if="order.length > 0">
								<div class=" py-3 px-0 m-0 relative border-t-2 border-solid border-slate-700 flex justify-between" v-for="order in order" :key="order.id">
									<div class="col-span-9 p-0 border-2 border-lime-100">
										<div class="flex w-56">
											<div class="w-14 h-14 bg-cover bg-center bg-no-repeat border-2 border-lime-100" v-bind:style="{ backgroundImage: 'url('+ order.image +')' }"></div>
											<div class="flex-1 ps-5">
												<div class="text-black text-sm">{{ order.title }}</div>
												<div class="single-price text-black">${{ order.price }}</div>
												<div class="desc text-black">
													<div v-for="option in order.options" :key="option.id">- {{ option.key }}: {{ option.value }}</div>
												</div>
												<div class="w-24 mt-3 items-center flex">
													<div class="input-group-append">
														<a href="#" class="btn btn-default border-2 border-lime-100 text-black flex-1 p-0" v-on:click="(event) => deductQty(event, order.id)"><i class="fa fa-minus"></i></a>
													</div>
													<input type="text" v-model="order.quantity" class=" m-0 bg-inherit border-none p-0 text-black text-center leading-none h-7 w-10 font-normal" />
													<div class="input-group-prepend">
														<a href="#" class="btn btn-default border-2 border-lime-100 text-black flex-1 p-0" v-on:click="(event) => addQty(event, order.id)"><i class="fa fa-plus"></i></a>
													</div>
												</div>
											</div>
										</div>
									</div>
									<div class="column-3 flex flex-col justify-between text-black">
										<div>${{ (order.price * order.quantity).toFixed(2) }}</div>
										<div class="text-end mt-auto"><a href="#" v-on:click="(event) => toggleConfirmation(event, order.id, true)" class="btn btn-default btn-sm p-0 border-none w-7 h-7 flex items-center justify-center"><i class="fa fa-trash"></i></a></div>
									</div>
									
									<div class="text-black pos-remove-confirmation absolute top-2 -left-3 -right-3 bottom-2 w-auto rounded-lg text-lg font-bold z-10 p-5 flex items-center " v-if="order.confirmation">
										<div class="text-center mx-auto">
											<div><i class="far fa-trash-can fa-2x text-body text-opacity-50"></i></div>
											<div class="mt-1 mb-2">Confirm to remove this item? </div>
											<div>
												<a href="#" v-on:click="(event) => toggleConfirmation(event, order.id, false)" class="btn btn-default w-60px me-2">No</a>
												<a href="#" v-on:click="(event) => removeOrder(event, order.id)" class="btn btn-danger w-60px">Yes</a>
											</div>
										</div>
									</div>
								</div>
							</div>
							<!-- END pos-table -->
							<div v-else class="h-100 d-flex align-items-center justify-content-center text-center p-20">
								<div>
									<div class="mb-3 mt-n5">
										<i class="fa fa-utensils text-body text-opacity-25" style="font-size: 5em"></i>
									</div>
									<h5>No order found</h5>
								</div>
							</div>
                       </div>
                    </div>
                   
                </div>
                
            </div>
        </div>
        
</template>
<style lang="scss">
@import '../assets/style.scss';

</style>
