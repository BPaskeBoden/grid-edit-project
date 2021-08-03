<template>
    <div>
        <b-form @submit="onSubmit" >
            <ul>
                <li v-for="styleOption in stockData.styleOptions">
                    <div> Option: <span class="boldName">{{styleOption.name}}</span></div>
                    <ul>
                        <li v-for="size in styleOption.sizes">
                            <div> Size: <span class="boldName">{{size.name}}</span></div>
                            <ul>
                                <li v-for="store in size.stores">
                                    <div class="row">
                                        <div class="col-6"> Store: <span class="boldName">{{store.name}}</span></div>
                                        <div class="col-6">
                                            <!--span><span class="qtybox">{{store.unit}}</span><b-form-input
                                                id="input-1"
                                                v-model="store.qty"    
                                                required>
                                            </b-form-input></span-->

                                             <b-form-checkbox
      id="checkbox-1"
      v-model="store.checked"
      name="checkbox-1"
      value="true"
      unchecked-value="false"
     
    >     
    </b-form-checkbox>

    
                                        </div>
                                    </div>
                                </li>
                            </ul> 
                        </li>
                    </ul>                    
                </li>
            </ul>

             <b-button id="btnSubmit" type="submit" variant="primary">Submit</b-button>
        </b-form>
    </div>
</template>

<script>
export default {
    name: 'ReplenishmentForm',
    data () {
        return {
        }
    },
    props: {
        stockData: {
        
        }
    },
    methods: {
      onSubmit(event) {
        event.preventDefault()
        alert(JSON.stringify(this.stockData))
      }
    }
}
</script>