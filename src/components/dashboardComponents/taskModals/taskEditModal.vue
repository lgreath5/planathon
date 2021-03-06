<template>
  <transition name="modal">
    <div class="popup-background">
      <div class="popup-wrapper" @click="$parent.showEditModal = -1">
        <!-- We use @click.stop on the next line to prevent showEditModal
        from being changed when clicking on the purple text -->
        <div class="popup-table purple-gradient" style="align: center" @click.stop>
          <h2>Edit Task</h2>
            <div style="display: flex" class="task-options">
              <div>
                <p>Title:</p>
                <input v-model="taskTitle" type="text" class="auth-textbox"
                      @keyup.enter=""/><br>
                <p>Description:</p>
                <input v-model="taskDesc" type="text" class="auth-textbox"
                      @keyup.enter=""/>
              </div>
              <div>
                <p>Tags:</p>
                <div class="tag-picker">
                  <div class="tag-toggle">
                    <toggle-button v-model="finance" color="#FFE16E"/>
                    <label for="checkbox">  Finance</label>
                  </div>
                  <div class="tag-toggle">
                    <toggle-button v-model="sponsors" color="#FF9A6E"/>
                    <label for="checkbox">  Sponsors</label>
                  </div>
                  <div class="tag-toggle">
                    <toggle-button v-model="promotion" color="#FF6EE0"/>
                    <label for="checkbox">  Promotion</label>
                  </div>
                  <div class="tag-toggle">
                    <toggle-button v-model="design" color="#BA68FF"/>
                    <label for="checkbox">  Design</label>
                  </div>
                  <div class="tag-toggle">
                    <toggle-button v-model="general" color="#7A8FFF"/>
                    <label for="checkbox"> General</label>
                  </div>
                </div>
              </div>
            </div>

            <p>Task Search</p>
            <input type="text" v-model="taskSearch" class="auth-textbox">
            </input><br>

            <div style="display: flex" class="task-options">
              <div>
                <p>All Tasks:</p>
                <select v-if="taskSearch == ''" v-model="taskSelect" class="auth-textbox" multiple>
                  <option v-for="task in tasks" :value="task">{{task.title}}</option>
                </select>
                <select v-if="!taskSearch == ''" v-model="taskSelect" class="auth-textbox" multiple>
                  <option v-for="result in taskResults" :value="result">{{result.title}}</option>
                </select>
              </div>
              <div>
                <p>Current Dependencies:</p>
                <select v-if="taskSearch == ''" v-model="depSelect" class="auth-textbox" multiple>
                  <option v-for="dep in taskDeps" :value="dep">{{dep.title}}</option>
                </select>
                <select v-if="!taskSearch == ''" v-model="depSelect" class="auth-textbox" multiple>
                  <option v-for="result in depResults" :value="result">{{result.title}}</option>
                </select>
              </div>
            </div>
            <br>

            <div style="display: flex" class="task-options">
              <button class="material-button-large" @click="addDeps()">Add Dependencies</button>
              <button class="material-button-large" @click="removeDeps()">Remove Dependencies</button>
            </div>
            <br>
            <div style="display: flex" class="task-options">
              <button class="material-button-large" @click="saveTask()">Save</button>
              <button class="material-button-large" @click="$emit('close')">Close</button>
            </div>
          </div>
        </div>
      </div>
  </transition>
</template>

<script>
  export default {
    data() {
      return {
        showEditModal: false,
        taskTitle: '',
        taskDesc: '',
        promotion: false,
        general: false,
        design: false,
        sponsors: false,
        finance: false,
        taskSearch: '',
        taskSelect: [],
        depSelect: [],
        newDepList: []
      }
    },
    props: {
      task: {
        type: Object,
        required: true
      }
    },
    mounted() {
      // Initialize variables for editing task tags
      if (this.task.tags.includes("promotion")) {
        this.promotion = true;
      }
      if (this.task.tags.includes("design")) {
        this.design = true;
      }
      if (this.task.tags.includes("general")) {
        this.general = true;
      }
      if (this.task.tags.includes("finance")) {
        this.finance = true;
      }
      if (this.task.tags.includes("sponsors")) {
        this.sponsors = true;
      }

      // These variables are used for reverting changes when cancelling and edit
      this.taskTitle = this.task.title;
      this.taskDesc = this.task.description;
    },
    methods: {
      // Makes the new list of dependencies, but won't save it until
      // saveTask() is called.
      // TODO - Check for cycles
      addDeps() {
        this.newDepList = this.taskDeps;
        this.taskSelect.forEach(task => {
          if (task.daysBefore >= this.task.daysBefore) {
            this.newDepList.push(task.id);
          }
        });
      },
      // Similar to addDeps, won't save until saveTask is called.
      removeDeps() {
        this.newDepList = this.taskDeps;
        this.depSelect.forEach(dep => {
          var index = this.newDepList.indexOf(dep);
          if (index !== -1) this.newDepList.splice(index, 1);
        });
      },
      saveTask() {
        var updatedTags = [];
        if (this.promotion) {
          updatedTags.push("promotion");
        }
        if (this.finance) {
          updatedTags.push("finance");
        }
        if (this.general) {
          updatedTags.push("general");
        }
        if (this.sponsors) {
          updatedTags.push("sponsors");
        }
        if (this.design) {
          updatedTags.push("design");
        }

        this.$store.dispatch('tasks/set', {[`${this.task.id}`]: {
          title: this.taskTitle,
          description: this.taskDesc,
          tags: updatedTags,
          dependencies: this.newDepList
        }})
        .then(() => {
         this.task.title = this.taskTitle;
         this.task.description = this.taskDesc;
         this.task.tags = updatedTags;
         this.$emit('close');
        }).catch(err => {
          this.$parent.$parent.$parent.$parent.messages.push("Could not edit task: " + err.message);
        })
      }
    },
    computed: {
      tasks() {
        return this.$store.getters['tasks/hackathonTasks'](this.task.hackathon)
      },
      taskDeps() {
        return this.$store.getters['tasks/taskDeps'](this.task.id)
      },
      taskResults() {
        return this.tasks.filter((task) => {
          return task.title.toLowerCase().includes(this.taskSearch.toLowerCase());
        })
      },
      depResults() {
        return this.taskDeps.filter((dep) => {
          return dep.title.toLowerCase().includes(this.taskSearch.toLowerCase());
        })
      }
    }
  }
</script>

<style scoped lang="scss">
@import '@/GlobalVars.scss';

/*  Note that this only applies to this page's material button: */
  .material-button-large {
    background: $gray;
    color: white;
  }

  .tag-picker {
    background: $gray;
    border-radius: 10px;
    padding: 10px;
    text-align: right;
  }
  .tag-toggle {
    display: flex;
    justify-content: space-between;
    width: 100%;
  }
  .task-options {
    display: flex;
    justify-content: space-between;
  }

  .auth-textbox {
    width: 190px;
  }
</style>
