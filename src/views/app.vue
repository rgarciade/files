<template>
  <v-app>
    <openfilesbutton principalClass="button-open-copidir"  :files="copiDirsArray"></openfilesbutton>
      <v-dialog
        v-model="repeatIDStatus"
        max-width="290"
      >
      <v-card>
          <v-card-title class="headline">existe un id de orden repetido</v-card-title>
          <v-card-text>
             id repetido: {{repeatID}}
          </v-card-text>
        </v-card>
      
      </v-dialog>
      <v-snackbar
        v-model="snackbar"
        buttom
      >
        {{ snackbarText }}
        <v-btn
          color="pink"
          flat
          @click="snackbar = false"
        >
          Cerrar
        </v-btn>
      </v-snackbar>

    <v-tabs fixed-tabs
      v-model="tabs"
    >
      <v-tab
        v-for=" tab in dirsToSee"
        :key="tab.name"
        :valu="tab"
      >
        {{ tab.name }}
      </v-tab>
    </v-tabs>
    <v-tabs-items class="item-file" v-model="tabs">
          <v-tab-item
            v-for=" tab in dirsToSee"
            :key="tab.id"
            :valu="tab"
          >
        <v-layout>
          <v-flex>
            <v-card-text>
              <v-treeview v-if="tab.files"
                v-model="filesSelected[tab.id]"
                :items="tab.files"
                activatable
                active-class="grey lighten-4 indigo--text"
                selected-color="indigo"
                selectable
                on-icon="bookmark"
                off-icon="bookmark"
              >
              </v-treeview>
            </v-card-text>
          </v-flex>
          <v-divider vertical></v-divider>
  
        <v-flex
          xs3
          class="lista-fechas"
        >
          <v-list
            v-for=" file in tab.files"
            :key="file.name"
            class="listaca "
          >
          <div class="filaca">
             {{ file.mtime }}
          </div>
             
          </v-list>
        </v-flex>
        <v-divider vertical></v-divider>
        <v-flex
          xs3
        >
          <v-card-text class="seleccionados">
            <div
              v-if="selections.length === 0"
              key="title"
              class="title font-weight-light grey--text pa-3 text-xs-center"
            >
             Ninguno seleccionado
            </div>
  
            <v-scroll-x-transition
              group
              hide-on-leave
            >
              <v-chip
                v-for="(selection, i) in selections"
                :key="i"
                color="grey"
                dark
                small
              >
                <v-icon left small>mdi-beer</v-icon>
                {{ selection.name }}
              </v-chip>
            </v-scroll-x-transition>
          </v-card-text>
        </v-flex>
        </v-layout>
        
      </v-tab-item>
    </v-tabs-items>
            <v-dialog
          v-model="dialog"
          fullscreen
          hide-overlay
          transition="dialog-bottom-transition"
          scrollable
        >
          <v-card>
            <v-toolbar card dark color="primary">
              <v-btn icon dark @click="dialog = 0">
                <v-icon>close</v-icon>
              </v-btn>
              <v-toolbar-title>Directorios</v-toolbar-title>
              <v-spacer></v-spacer>
              <input id="newCopyDir" type="file" webkitdirectory style="display: none" />
              <v-toolbar-title v-if="DirsConfig.length < 1" class="animation-nueva">
                agregar carpetas  <v-icon >arrow_right_alt </v-icon>
              </v-toolbar-title>
                <v-toolbar-items>
                        <v-tooltip bottom>
                          <template v-slot:activator="{ on }">
                            <v-btn dark flat  @click="addNewDir" v-on="on">
                              <input id="newDir" type="file" webkitdirectory style="display: none" />
                              
                                <v-icon>create_new_folder</v-icon>
                            </v-btn>
                          </template>
                          <span>Añadir Carpeta</span>
                        </v-tooltip>  
                </v-toolbar-items>
                <v-toolbar-items>
                  <v-btn dark flat @click="checkPasswordExist">Guardar</v-btn>
                </v-toolbar-items>
            </v-toolbar>
            <v-flex v-if="DirsConfig.length < 1" sm6 offset-sm3 class="initial-card">
                <v-card> 
                  <v-card-title primary-title>
                    <div>
                      <h3 class="headline mb-0">Seleccione las carpetas para comenzar</h3>
                    </div>
                  </v-card-title>
                </v-card>
              </v-flex>
            <v-list two-line>
            <template v-for="(item, index) in DirsConfig">
              <v-layout row :key="index">
                <v-flex xs2 >
                  <v-select 
                    :items="orden"
                    v-model="item.id"
                    placeholder="cambiar"
                    suffix="Orden"
                    solo-inverted
                          >
                  </v-select>
                </v-flex>
                <v-flex xs8>
                    <v-list-tile :key="index" avatar ripple>
                      <v-list-tile-content>
                        <v-list-tile-title>
                          Orden: {{ item.id }}
                          </v-list-tile-title>
                        <v-list-tile-sub-title class="text--primary">Nombre de la carpeta: {{ item.name  }}</v-list-tile-sub-title>
                        <v-list-tile-sub-title>Dirección a la carpeta: {{ item.url }}</v-list-tile-sub-title>
                      </v-list-tile-content>
                      <v-list-tile-action>
                        <v-tooltip left>
                          <template v-slot:activator="{ on }">
                            <v-icon color="grey lighten-1" v-on="on" @click="removeDir(item.url)">delete_forever</v-icon>
                          </template>
                            <span>Quitar del programa</span>
                        </v-tooltip>   
                      </v-list-tile-action>
                    </v-list-tile>
                    <v-divider v-if="index < (DirsConfig.length -1)"  :key="`divider-${index}`" ></v-divider>
                  </v-flex>
                  <v-flex xs8>
                    <v-list-tile :key="index" avatar ripple>
                      <v-list-tile-content>
                        <v-list-tile-title>
                          Dirección de copia 
                          </v-list-tile-title>
                        <v-list-tile-sub-title> {{(item.copydir && item.copydir != '')? item.copydir: "No asignado" }} </v-list-tile-sub-title>
                      </v-list-tile-content>
                      <v-list-tile-action>
                        <v-tooltip left>
                          <template v-slot:activator="{ on }">
                            <v-icon v-if="item.copydir && item.copydir != ''"  color="grey lighten-1" v-on="on" class="clea-Copy-directory"  @click="removeCopyFile(item.id)">clear</v-icon>
                          </template>
                          <span>quitar copia  automatica</span>
                        </v-tooltip>   
                        <v-tooltip left>
                          <template v-slot:activator="{ on }">
                            <v-icon color="grey lighten-1" v-on="on" class="arrow"  @click="selectCopyFile(item.id)">label_important</v-icon>
                          </template>
                          <span>Copia automatica</span>
                        </v-tooltip>   
                      </v-list-tile-action>
                    </v-list-tile>
                    <v-divider v-if="index < (DirsConfig.length -1)"  :key="`divider-${index}`" ></v-divider>
                  </v-flex>
              </v-layout>
            </template>
          </v-list>
          </v-card>
        </v-dialog>
        <div class='element-add'>

          <v-tooltip top>
            <template v-slot:activator="{ on }">
              <v-btn color="blue" class='button-add' v-on="on" fab @click="dialog = true">
                <v-icon>add_box</v-icon>
              </v-btn>
            </template>
            <span>Configuración</span>
          </v-tooltip>

        </div>
        <div class='element-recharge'>
          <v-tooltip top>
            <template v-slot:activator="{ on }">
              <v-btn color="primary" class='button-add' v-on="on"  fab @click="prepareDisrsAndItemsDirs()">
                <v-icon>replay</v-icon>
              </v-btn>
            </template>
            <span>Recarga</span>
          </v-tooltip>   
        </div>
  
      <v-dialog
        v-model="dialogCheck"
        width="500">
        <template v-slot:activator="{ on }">
                <div class='pasar-archivo'>
          <v-tooltip bottom>
              <template v-slot:activator="{ on }">
                <v-btn small flat @click="openfocusDir"><v-icon>folder</v-icon></v-btn>
              </template>
              <span>Abrir Carpeta</span>
          </v-tooltip>   
          <v-btn  small color="success" @click="openFiles">abrir Archivos</v-btn>
          <v-btn  small color="info" v-on="on">Pasar Archivos</v-btn>
          <v-scroll-y-transition>
          <v-btn v-if="(dirsToSee[tabs] && dirsToSee[tabs]['copydir'])" small color="green" @click="openCopyDir()">Abrir {{nameCopyDir}}</v-btn>
          </v-scroll-y-transition>
        </div>
        </template>
        <v-card>
          <v-card-title
            class="headline grey lighten-2"
            primary-title
          >
            Confirmación
          </v-card-title>
          <v-card-text>
            ¿Confirma que quiere transpasar el archivo?
          </v-card-text>
          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="red"
              flat
              @click="dialogCheck = false">
              cancelar
            </v-btn>           
            <v-btn
              color="primary"
              flat
              @click="moveFiles">
              transpasar
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-dialog
        v-model="CheckNewPasswordStatus"
        width="500">
        <template v-slot:activator="{ on }">  
          <v-btn small flat @click="" id="chekpass" style="display: none"><v-icon></v-icon></v-btn>
        </template>
        <v-card>
          <v-card-title
            class="headline grey lighten-2"
            primary-title
          >
            Nueva contraseña
          </v-card-title>
          <v-card-text>
            Introduce la nueva contraseña
                        <v-text-field
              v-model="firstPass"
              label="Contraseña"
              required
            ></v-text-field>
            <v-text-field
              v-model="seconfPass"
              label="Repita la contraseña"
              required
            ></v-text-field>
          </v-card-text>

          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="primary"
              flat
              @click="createPass">
              Crea Contraseña
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-dialog
        v-model="CheckPasswordStatus"
        width="500">
        <template v-slot:activator="{ on }">  
          <v-btn small flat @click="" id="chekpass" style="display: none"><v-icon></v-icon></v-btn>
        </template>
        <v-card>
          <v-card-title
            class="headline grey lighten-2"
            primary-title
          >
            comprueba contraseña
          </v-card-title>
          <v-card-text>
            Comprueba la contraseña
                        <v-text-field
              v-model="compobarPass"
              label="Contraseña"
              required
            ></v-text-field>
          </v-card-text>

          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="primary"
              flat
              @click="checkUpdatePassword">
              actualizar datos
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

  </v-app>
</template>

<script>
  import { getFiles, moveFileToNewDir, opendir, openFile, getDirs, checkPasswordExistInTheProgram, insertNewPass, checkPassword } from "../commonFunctions.js";
  import dataStorage from '../components/datastorage'
  import { remote } from 'electron'
  let app  = {
      name: 'app',
      data(){
          return { 
            tabs:0,
            tree:[],
            dialog: false,
            dirs:[],
            DirsConfig:[],
            orden:[],
            repeatIDStatus: false,
            repeatID: 0,
            snackbar: false,
            snackbarText: '',
            dirsToSee:[],
            filesSelected:[],
            copyDirId:null,
            dialogCheck:false,
            CheckNewPasswordStatus:false,
            CheckPasswordStatus:false,
            firstPass:'',
            seconfPass:'',
            compobarPass:'',
            nameCopyDir:''
          }
      },
      mounted(){    
        let that = this
        document.getElementById('newDir').addEventListener('change', e => {
          let dirName = e.target.files[0].name
          let realUri = e.target.files[0].path
          this.addToDirToStorage({'name':dirName,'url':realUri})
          document.getElementById('newDir').value = ''
        })
        document.getElementById('newCopyDir').addEventListener('change', e => {
          let dirName = e.target.files[0].name
          let realUri = e.target.files[0].path
          this.addCopyToDirToStorage(realUri)
          document.getElementById('newCopyDir').value = ''
        })
        setInterval(function(){ 
          that.prepareDisrsAndItemsDirs()
        }, 4000);
        this.prepareDisrsAndItemsDirs()
      },
      methods:{
        checkUpdatePassword(){
          checkPassword(this.compobarPass).then(resp =>{
            this.sabeDirConfif()
            this.activeSnackbar('directorios actualizados')
          }).catch(()=>this.activeSnackbar('error con la contraseña'))
        },
        createPass(){
          if(this.firstPass == this.seconfPass){
            insertNewPass(this.firstPass,this.seconfPass).then(status =>{
                this.CheckNewPasswordStatus = false
                this.sabeDirConfif()
                this.activeSnackbar('contraseña creada')
            }).catch(err => this.activeSnackbar('ocurrió un error al crear la contraseña'))
          }else{
            this.activeSnackbar('las contraseñas no son iguales')
          }
        },
        checkPasswordExist(){
          checkPasswordExistInTheProgram().then(resp =>{
            if(!resp){
              //no existepass, crear
              this.CheckNewPasswordStatus = true
            }else{
              //existe pass, comprobar
              this.CheckPasswordStatus = true
            }
          })
        },
        getAbsoluteUrlFile(fileName){
          return this.dirsToSee[this.tabs].url +'/'+fileName
        },
        getConfigDirs(){
          let tempComf = []
          for (let index = 0; index < this.DirsConfig.length; index++) {
            tempComf[index] = this.DirsConfig[index]; 
          }
          return tempComf
        },
        closeDialogCheck(){
          if(this.dirsToSee.length >= 1){
            this.dialog = true
          }else{
            this.dialog = false
          }
        },
        async prepareDisrsAndItemsDirs(change = false) {
          let dirs = await getDirs()
          
           for (let index = 0; index < dirs.length; index++) {
            const element = dirs[index];
            await getFiles(element.url)
            .then(files=> {
              dirs[index].files = files
              dirs[index].id = index
            }) 
          } 
          if(dirs[0] && change){
            let newTemp = { 'name': '','id':dirs.length, 'files': [{id:999,name:'',url:''}]}
            dirs[dirs.length] = newTemp
          }
          
          this.dirsToSee = dirs
          
          if(change == 1){
            setTimeout(function(){ dirs.splice(-1,1);this.dirsToSee = dirs }, 1);
          }
        },

        removeDir(url){
          let thisDirs = this.getConfigDirs()
          for(var i = thisDirs.length - 1; i >= 0; i--) {
            if(thisDirs[i].url === url){
                  thisDirs.splice(i, 1);
            }
          }
          for(var i = thisDirs.length - 1; i >= 0; i--) {
            thisDirs[i].id = i
          }
          this.DirsConfig = thisDirs
        },
        updateDirs(thisDirs){
          this.Dirs = thisDirs

          this.orden = []
          for (let index = 0; index < this.Dirs.length; index++) {
            this.orden.push(index)
          }
          dataStorage.getVal('nombre', '=', 'dirs').then(dataStorageData =>{
            if(dataStorageData[0]){
              dataStorage.updateVals('nombre', '=', 'dirs', { "datos": thisDirs }).then(a => console.log(a))
            }else{
              dataStorage.setVal({ "nombre": "dirs", "datos": thisDirs  })
            }
            this.activeSnackbar('Directorios actualizados')
            this.dialog = false
            location.reload();
          })
        },
        activeSnackbar(text){
          this.snackbar = true
          this.snackbarText = text
        },
        sabeDirConfif(){
          let newDirs = this.getConfigDirs()
          let newDirsOrder = []
          
          for (let index = 0; index < newDirs.length; index++) {
            const id = newDirs[index].id
            let sumid = 0 
            for (let a = 0; a < newDirs.length; a++) {
              if(id == newDirs[a].id){
                sumid ++
              }
              if(sumid > 1){
                a = newDirs.length
                index = newDirs.length
                this.repeatIDStatus = true
                this.repeatID = id
              }
            }
          }
          for (let index = 0; index < newDirs.length; index++) {
            newDirsOrder[newDirs[index].id] = newDirs[index]
          }
          if(!this.repeatIDStatus){
            this.updateDirs(newDirsOrder)
          }
        },
        addNewDir(event){
          document.getElementById('newDir').click()
        },
        selectCopyFile(id){
          this.copyDirId = id
          document.getElementById('newCopyDir').click()
        },
        removeCopyFile(id){
          this.copyDirId = id
          this.removeCopyToDirToStorage()
        },
        addToDirToStorage(newDir){
          let thisDirs = this.getConfigDirs()
          thisDirs[thisDirs.length] = { 'id':thisDirs.length, 'name': newDir.name, 'url': newDir.url}
          this.DirsConfig = thisDirs
          this.updateOrder()
        },
        addCopyToDirToStorage(url){
          let thisDirs = this.getConfigDirs()
          if(this.copyDirId != null){
            thisDirs[this.copyDirId].copydir = url
            this.DirsConfig = thisDirs
            this.updateOrder()
          }
        },
        removeCopyToDirToStorage(){
          let thisDirs = this.getConfigDirs()
          if(this.copyDirId != null){
            thisDirs[this.copyDirId].copydir = ''
            this.DirsConfig = thisDirs
            this.updateOrder()
          }
        },
        updateOrder(){
          if(this.getConfigDirs()){
            let tempDirs = this.getConfigDirs()
            
            this.orden = []
            for (let index = 0; index < tempDirs.length; index++) {
              this.orden.push(index)
            }
          }
        },
        getSelection(tabId){
          const selections = []
          if(this.filesSelected[this.tabs]){
            let files = this.dirsToSee[this.tabs].files
            for (let index = 0; index < this.filesSelected[this.tabs].length; index++) {
              const element = this.filesSelected[this.tabs][index];
              const file = files[element]
              selections.push(file)
            }
          }
          return selections
        },
        openFiles(){
          let filesSelecteds = this.getSelection(this.tabs)
          if(this.dirsToSee[this.tabs]){
            for (let index = 0; index < filesSelecteds.length; index++) {
              const selected = filesSelecteds[index];
              try {
                openFile(selected.url)
              } catch (error) {
                console.log(error)
              }
            }
          }   
        },
        openCopyDir(){
          if(this.dirsToSee[this.tabs]['copydir'] && this.dirsToSee[this.tabs]['copydir'] !=''){
            let dirUrl = this.dirsToSee[this.tabs]['copydir']
            openFile(this.dirsToSee[this.tabs]['copydir'])
          }
        },
        openfocusDir(){
          let destinationDirectory= this.dirsToSee[this.tabs].url
              try {
                openFile(destinationDirectory)
              } catch (error) {
                console.log(error)
              }
        },
        moveFiles (){
          this.dialogCheck = false; 
          let filesSelecteds = this.getSelection(this.tabs)
          if(this.dirsToSee[this.tabs + 1]){
            let destinationDirectory= this.dirsToSee[this.tabs + 1 ].url
            let promises = [];
            for (let index = 0; index < filesSelecteds.length; index++) {
              const selected = filesSelecteds[index];
              promises.push(moveFileToNewDir(destinationDirectory, selected.name, selected.url))
            }
            Promise.all(promises)
              .then(resp => {
                console.log('entra1')
                this.prepareDisrsAndItemsDirs()
              })
              .catch(err => {
                console.log('entra2')
                  console.log(err)
                  this.activeSnackbar('error al copiar el archivo'+filesSelecteds[index].name)
              })
            this.filesSelected = []
            
          }else{
            this.activeSnackbar('No existe proximo Directorios')
          }
        }        
      },
      computed:{
        selections () {
          return this.getSelection(this.tabs)
        },
        copiDirsArray(){
          let dirsReturn = []
          for (let index = 0; index < this.dirsToSee.length; index++) {
            const element = this.dirsToSee[index];
            if(element.copydir){
              const dirSeparator = ( remote.getGlobal('platform') == "win32" )? '\\' :'/'
              let name = element.copydir.substring(element.copydir.lastIndexOf(dirSeparator)+1);
              
              dirsReturn.push({
                'urlDir':element.copydir,
                'name':name
              })
            }
          }
          return dirsReturn
        }
      },
      watch: {
        dirsToSee: function (newDirsToSee){
          if(newDirsToSee.length < 1 ){
            this.dialog = true
          }
        },
        tabs: function (){
            if(this.dirsToSee[this.tabs]['copydir'] && this.dirsToSee[this.tabs]['copydir'] !=''){
              let dirUrl = this.dirsToSee[this.tabs]['copydir']
              const dirSeparator = ( remote.getGlobal('platform') == "win32" )? '\\' :'/'
              let name = dirUrl.substring(dirUrl.lastIndexOf(dirSeparator)+1);
              this.nameCopyDir = name
            }
        },
        dialog: async function (newDialog){
          if(!newDialog && this.dirsToSee.length == 0){
            this.dialog = 1
          }
          if(newDialog){
              this.DirsConfig = await getDirs()
              this.updateOrder()
          }
        }
    },
  }
  export default app
</script>
<style>
</style>
