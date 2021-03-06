<template>
  <div class="app-wrapper">

    <div class="columns">
        <div class="column is-12" v-if="msg">
           <div v-if="msg_status==200" class="notification is-primary">{{msg}}</div>
           <div v-if="msg_status!=200" class="notification is-danger">{{msg}}</div>
        </div>
    </div>

    <div class="columns">
      <div class="column is-5">
        <div class="app-heading heading has-text-centered">
          <p style="font-weight: bold;font-size: 16px;text-decoration: underline;">Toolbar</p>
        </div>
        <toolbar :draggable="components" :dropped="dropped"></toolbar>
        <!-- <p class="app-control">
          <label class="checkbox">
            <input type="checkbox" v-model="autoEdit">
            Show edit page after adding component
          </label>
        </p>
        <p class="app-control">
          <label class="checkbox">
            <input type="checkbox" v-model="autoSave">
            Automatically save template
          </label>
        </p> -->
        <p> 
           <button style="float: right;" class="button is-info" @click="savePreviewHTML">Save Preview</button>
           <a style="float: right; margin-right: 10px;" class="button is-primary" href="/marketing?tab=1">Back</a>
           <!-- <a class="button is-success is-right" href="/mail-content-view-html">View HTML</a>  -->
        </p>
      </div>
      <div class="column is-7">
        <div class="app-heading heading has-text-centered">
          <p style="font-weight: bold;font-size: 16px;text-decoration: underline;">Preview</p>
        </div>
        <preview @updatedSubject="createdSubject" :dropped="dropped"></preview>      
      </div>    
       
    </div>

  </div>
</template>

<script>

import Bus from './assets/js/EventBus'
import Toolbar from './Toolbar.vue'
import Preview from './Preview.vue'

export default {
  name: 'App',
  components: {
    Toolbar,
    Preview
  },
  data ()
  {
    return {
      subject:"",
      msg: "",
      msg_status:200,
      dropped: [],
      autoEdit: true,
      autoSave: true,
      components: {
        // the types of drop-in components and their default settings
        text: { 
          key: 'text',
          name: 'text', 
          justify: 'left',
          marginTop: 'none',
          marginBottom: 'none',
          fontSize: 'medium',
          color: 'black',
          text: 'Your text goes here...',
          bold: false,
          italic: false,
        },
        image: {
          key: 'image',
          name: 'image',
          justify: 'centered',
          marginTop: 'a little',
          marginBottom: 'a little',
          caption: '',
          height: '300',
          src: 'http://placehold.it/550x300',
        },
        button: { 
          key: 'button',
          name: 'button',
          justify: 'centered',
          marginTop: 'a little',
          marginBottom: 'a little',
          label: 'Click me!',
          color: 'blue',
        },
        // links: { 
        //   key: 'links',
        //   name: 'social links',
        //   marginTop: 'some',
        //   marginBottom: 'none',
        //   frozen: true, // no resizing this component
        //   links: ['fa-facebook', 'fa-twitter', 'fa-instagram'],
        // },
        divider: {
          key: 'divider',
          name: 'divider',
          marginTop: 'none',
          marginBottom: 'none',
        },
        space: {
          key: 'space',
          name: 'space',
          marginTop: 'a little',
          marginBottom: 'a little',
        },
      },
    }
  },

  /**
   * Attach listeners that handle state manipulation
   */
  mounted()
  {
   
    this.getPreviewHTML();

    Bus.listen('save', () => { this.save() });
    Bus.listen('component-dropped', (event) => { this.addComponent(event); this.save() });
    Bus.listen('destroy-component', (index) => { this.destroy(index); this.save() });
    Bus.listen('edit-component', (index) => { this.edit(index); this.save() });
    Bus.listen('clone-component', (index) => { this.clone(index); this.save() });
    Bus.listen('create-dropzone', (index) => { this.createDropzoneNextToComponent(index); this.save() });
    Bus.listen('destroy-dropzone', (index) => { this.destroyDropzoneNextToComponent(index); this.save() });
  },

  watch:
  {
    autoSave(val)
    {
      if (val) {
        this.save();
      }
      else {
        localStorage.setItem('autoSave', JSON.stringify(false));
        this.deleteStorage();
      }
    },

    autoEdit()
    {
      this.save();
    },
  },

  methods:
  {
    createdSubject(input){   
      this.subject =  input;
    },
    /**
     * Check for local browser storage
     */
    load()
    {
      this.dropped = localStorage.getItem('dropped') ? JSON.parse(localStorage.getItem('dropped')) : [];
      this.autoSave = localStorage.getItem('autoSave') ? JSON.parse(localStorage.getItem('autoSave')) : true;
      this.autoEdit = localStorage.getItem('autoEdit') ? JSON.parse(localStorage.getItem('autoEdit')) : false;
     
    },

    save()
    {
      if (this.autoSave) {
        localStorage.setItem('dropped', JSON.stringify(this.dropped));
        localStorage.setItem('autoSave', JSON.stringify(this.autoSave));
        localStorage.setItem('autoEdit', JSON.stringify(this.autoEdit));
      }
    },


    deleteStorage()
    {
      localStorage.removeItem('dropped');
      localStorage.removeItem('autoEdit');
    },

    /**
     * Add a component to the template depending on the info from the InteractJS event
     *
     * @param {object} event  The drop event that occurred
     */
    addComponent(event)
    {
      let index = parseInt(event.target.getAttribute('data-index')); // which index it was dropped in 
      let key = event.relatedTarget.getAttribute('data-key'); // what type of component was dropped
      let isSibling = event.target.getAttribute('sibling') != null;

      if (isSibling) {
        this.dropped[index].hasDropzone = false;
        let temp = JSON.parse(JSON.stringify(this.dropped[index])) // make copy
        temp.sibling = this.components[key]
        this.$set(this.dropped, index, temp);
      }
      else {
        this.dropped.splice(index, 0, JSON.parse(JSON.stringify(this.components[key])));
        this.dropped[index].index = index;
      }

      Bus.fire('component-added', index);
      Bus.fire('highlight-container', index);

      if (this.autoEdit) {
        Bus.fire('editing-component', { index: index, isSibling: isSibling });
      }
    },

    destroy(data)
    {
      if (data.isSibling) {
        // remove the sibling from the row
        let temp = JSON.parse(JSON.stringify(this.dropped[data.index])) // make copy
        temp.sibling = undefined;
        this.$set(this.dropped, data.index, temp);
      }
      else if (this.dropped[data.index].sibling) {
        // set sibling as only component in row
        let temp = JSON.parse(JSON.stringify(this.dropped[data.index].sibling)) // make copy
        this.$set(this.dropped, data.index, temp);
      }
      else {
        this.dropped.splice(data.index, 1);
      }
    },

    edit(data)
    {
      Bus.fire('editing-component', { index: data.index, isSibling: data.isSibling });
    },

    clone(data)
    {
      let newIndex = data.index + 1;
      if (data.isSibling) {
        let copy = JSON.parse(JSON.stringify(this.dropped[data.index].sibling))
        copy.index = newIndex
        this.dropped.splice(newIndex, 0, copy);
      }
      else {
        let copy = JSON.parse(JSON.stringify(this.dropped[data.index]))
        copy.sibling = undefined
        copy.index = newIndex
        this.dropped.splice(newIndex, 0, copy);
      }

      Bus.fire('component-added', { index: newIndex, isSibling: data.isSibling });
      Bus.fire('highlight-container', newIndex);

      if (this.autoEdit) {
        Bus.fire('editing-component', { index: newIndex, isSibling: data.isSibling });
      }
    },

    createDropzoneNextToComponent(index)
    {
      if (this.dropped[index].hasDropzone || this.dropped[index].sibling) {
        // don't create dropzones if there's something already there
        Bus.fire('highlight-container', index); // show some feedback about sizing
        return
      }

      let temp = JSON.parse(JSON.stringify(this.dropped[index])) // make non-reactive copy
      temp.hasDropzone = true;

      this.$set(this.dropped, index, temp);
      Bus.fire('highlight-container', index);
    },

    destroyDropzoneNextToComponent(index)
    {
      let temp = JSON.parse(JSON.stringify(this.dropped[index])) // make non-reactive copy
      temp.hasDropzone = false;

      this.$set(this.dropped, index, temp);
    },
    savePreviewHTML(){

      let _data = {
        content:this.dropped,
        subject:this.subject
      }      
      this.$http.post("/api/v1/mail-content/"+this.$route.params.id, _data)
          .then(response => {
           
            this.msg = response.data.msg;          
            this.msg_status = response.status;
            this.ajaxRequest = false;
          }).catch((err) => {
            if(err.data.msg){
              this.msg = err.data.msg;
            }else{
              this.msg = err.statusText;
            }        
            this.msg_status = err.status;
          });

    }, getPreviewHTML(){ 
         this.$http.get("/api/v1/mail-content/"+this.$route.params.id)
          .then(response => {            
            if(response.status==200){
              if(response.data.data.content){
                localStorage.setItem('dropped', response.data.data.content);
                Bus.fire('updated_subject', response.data.data.subject);
              }              
            }
            this.load();           
          }).catch((err) => {});
    },
  },
}
</script>

<style lang="sass">
@import '../node_modules/bulma/bulma'
@import './assets/sass/library'

.app-wrapper
  margin: 0 auto
  max-width: 1000px
  margin-top: 0px;
  min-height: 900px

.app-heading
  margin-bottom: 15px
  .title
    color: $blue

.app-control
  padding-left: 15px
  padding-bottom: 10px
  background: white
  &:not(:last-child)
    margin: 0
</style>
