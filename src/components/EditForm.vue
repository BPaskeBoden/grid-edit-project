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
                                            <span><b-form-input
                                                id="input-1"
                                                v-model="store.qty"    
                                                required>
                                            </b-form-input></span>
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
        <!--b-form @submit="onSubmit" >
            <b-form-group
                id="input-group-1"
                label="SKU:"
                label-for="input-1"              
            >
                <b-form-input
                    id="input-1"
                    v-model="stockData[0].sku"    
                    required>
                </b-form-input>
            </b-form-group>
            <b-form-group
                id="input-group-2"
                label="Store:"
                label-for="input-2"              
            >
                <b-form-input
                    id="input-2"
                    v-model="stockData[0].store"    
                    required>
                </b-form-input>
            </b-form-group>
             <b-form-group
                id="input-group-3"
                label="Style Option:"
                label-for="input-3"              
            >
                <b-form-input
                    id="input-3"
                    v-model="stockData[0].style"    
                    required>
                </b-form-input>
            </b-form-group>
           

        </b-form-->
      
    </div>
</template>

<script>
export default {
    name: 'EditForm',
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
<style>
  #btnSubmit {
    position: relative;
    top: 70px;
  }

li {list-style-type: none;}

.boldName {
    font-weight: bold;
}

.form-control {
    display: inline-block !important;
    width: 20% !important;
    padding: 1px 0px 1px 3px!important;
}
</style>