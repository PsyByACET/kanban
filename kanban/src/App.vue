<template>
  <div class="container mt-4">
    <div class="row mb-2">
      <b-form class="col-sm-12 col-md-6">
        <div class="form-inline mb-2">
          <b-form-input v-model="newTask" class="add-text" placeholder="Введите задачу" @keyup.enter="add" />
          <b-button class="ml-sm-2 mt-2 mt-sm-0 mr-4 but-add"  @click="add">Добавить</b-button>
        </div>
      </b-form>

      <div class="text-md-right col-sm-12 col-md-6 mb-2">
       <b-form-checkbox
          v-model="darkChecked"
          name="check-button"
          @change="toggleStyleLink()"
          switch>
          <span v-if="!darkChecked">Светлая тема</span>
          <span v-if="darkChecked">Темная тема</span>
        </b-form-checkbox>
        <h1 class="text-md-right">Kanban board </h1>
      </div>
    </div>

    <div class="row mt-3">
      <!-- 1 column -->
      <div class="col-md-4">
        <div class="p-2 main-colomn">
          <h2>Список задач ({{column[0].length}})</h2>

          <draggable class="list-group" :list="column[0]" group="tasks" :move="setTime">
            <div
              class="list-group-item container dark"
              v-for="(element, index) in column[0]"
              :key="element.task">
              <div class="row mb-3">
                <h4 class="col mb-0 h4">Задача №{{ element.id + 1}}</h4>
                <!-- Edit button -->
                <b-button
                  class="pl-2 pr-2 mr-2"
                  variant="warning"
                  v-b-modal="'my-modal'"
                  @click="setChosenElement(0, index)"
                  v-b-tooltip.hover
                  title="Редактировать">
                  💠
                </b-button>
                <!-- Button next column -->
                <b-button
                  class="pl-2 pr-2 btn-next"
                  variant="success"
                  @click="moveToNextColomn(0, index)"
                  v-b-tooltip.hover
                  title="Переместить в следущую колонку">
                  ✔
                </b-button>
              </div>

              <!-- Task -->
              <div class="row task">
                <p class="col text-truncate mb-2 h5">{{ element.task }}</p>
              </div>
            </div>
          </draggable>
        </div>
      </div>

      <!-- 2 column -->
      <div class="col-md-4">
        <div class="p-2 main-colomn">
          <h2>В процессе ({{column[1].length}})</h2>
          <draggable class="list-group" :list="column[1]" group="tasks" :move="setEndTime">
            <div
              class="list-group-item container dark"
              v-for="(element, index) in column[1]"
              :key="element.task">
              <div class="row mb-3">
                <!-- Title -->
                <h4 class="col mb-0 h4">Задача №{{ element.id + 1 }}</h4>

                <!-- Edit button -->
                <b-button
                  class="pl-2 pr-2 mr-2"
                  variant="warning"
                  v-b-tooltip.hover
                  title="Редактировать"
                  @click="setChosenElement(1, index)"
                  v-b-modal="'my-modal'">
                  💠
                </b-button>

                <!-- Button move to next column -->
                <b-button
                  class="pl-2 pr-2"
                  variant="success"
                  @click="moveToNextColomn(1, index)"
                  v-b-tooltip.hover
                  title="Переместить в следущую колонку">
                  ✔
                </b-button>
              </div>

              <!-- Task -->
              <div class="row task">
                <p class="col text-truncate mb-2 h5">{{ element.task }}</p>
              </div>
              <div class="row worker" v-if="element.worker">
                <p class="col mb-0">
                  <u>Ответственный:</u>
                  {{ element.worker }}
                </p>
              </div>
              <div class="row date">
                <p class="col mb-0">
                  <u>Дата начала:</u>
                  {{ element.dt.toLocaleString() }}
                </p>
              </div>
            </div>
          </draggable>
        </div>
      </div>

      <!-- 3 column -->
      <div class="col-md-4">
        <div class="p-2 main-colomn">
          <h2>Готово ({{ column[2].length }})</h2>

          <draggable class="list-group" :list="column[2]" group="tasks">
            <div
              class="list-group-item container dark"
              v-for="(element, index) in column[2]"
              :key="element.task">
              <div class="row mb-3">

                <h4 class="col mb-0 h4">Задача №{{ element.id + 1 }}</h4>
                <!-- Edit button -->
                <b-button
                  class="pl-2 pr-2 mr-2"
                  variant="warning"
                  v-b-tooltip.hover
                  title="Редактировать"
                  @click="setChosenElement(2, index)"
                  v-b-modal="'my-modal'">
                  💠
                </b-button>

                <!-- Button delete-->
                <b-button
                  class="pl-2 pr-2"
                  variant="danger"
                  @click="delCard(2, index)"
                  v-b-tooltip.hover
                  title="Удалить карточку">
                  ❌
                </b-button>
              </div>

              <!-- Task -->
              <div class="row task">
                <p class="col text-truncate mb-2 h5">{{ element.task }}</p>
              </div>
              <div class="row worker" v-if="element.worker">
                <p class="col mb-0">
                  <u>Ответственный:</u>
                  {{ element.worker }}
                </p>
              </div>
              <div class="row date">
                <p class="col mb-0">
                  <u>Дата начала:</u>
                  {{ element.dt.toLocaleString() }}
                </p>
              </div>
              <div class="row date-end">
                <p class="col mb-0">
                  <u>Дата окончания:</u>
                  <span>{{ element.endDt.toLocaleString() }}</span>
                </p>
              </div>
            </div>
          </draggable>
        </div>
      </div>
    </div>
     <!-- The modal window-->
    <b-modal id="my-modal" @ok="saveButton()">
      <b-form>
        <b-form-group id="input-group-2" label="Ваше задание" label-for="input-2">
          <b-form-input id="input-2" v-model="changed.task" required placeholder="Задача"></b-form-input>
        </b-form-group>

        <label class="mr-sm-2" for="select-pref">Статус</label>
        <b-form-select
          id="select-pref"
          class="mb-2 mr-sm-2 mb-sm-0"
          :options="['Список задач', 'В процессе', 'Готово']"
          v-model="changed.status"></b-form-select>

        <b-form-group
          v-if="changed.status == 'В процессе' || changed.status == 'Готово'"
          id="input-group-3"
          label="Ответственный"
          label-for="input-3">
          <b-form-input
            id="input-3"
            v-model="changed.worker"
            required
            placeholder="Enter worker name"></b-form-input>
        </b-form-group>
        <b-form-group
          v-if="changed.status == 'В процессе' || changed.status == 'Готово'"
          id="input-group-4"
          label="Выберите дату начала"
          label-for="datepicker">
          <date-picker
            id="datepicker"
            v-model="changed.dt"
            type="datetime"
            placeholder="Select date and time"
          ></date-picker>
        </b-form-group>

        <b-form-group
          v-if="changed.status == 'Done'"
          id="input-group-5"
          label="Choose end date"
          label-for="datepicker-end">
          <date-picker
            id="datepicker"
            v-model="changed.endDt"
            type="datetime"
            placeholder="Select date and time"
          ></date-picker>
        </b-form-group>
      </b-form>
    </b-modal>
  </div>
</template>

<script>
import draggable from "vuedraggable";
import DatePicker from "vue2-datepicker";
import "vue2-datepicker/index.css";

export default {
  name: "App",
  components: {
    draggable,
    DatePicker
  },

  data() {
    return {
      newTask: "",
      darkChecked: false,
      changed: {
        id: 0,
        column: 0,
        task: "",
        status: "",
        worker: "",
        dt: new Date(),
        endDt: new Date()
      },
      countTasks: 2,
      column: [
        [
          {
            id: 0,
            task: "Сделать дз",
            worker: "SAnya",
            dt: new Date(),
            endDt: new Date()
          }
        ],
        [
          {
            id: 1,
            task: "Делаю дз ",
            worker: "Alexey",
            dt: new Date(),
            endDt: new Date()
          }
        ],
        [
          {
            id: 2,
            task: "Чет сложна, лучше доту",
            worker: "Alexey",
            dt: new Date(),
            endDt: new Date()
          }
        ]
      ]
    };
  },

  methods: {
    // Add new task
    add() {
      if (this.newTask) {
        this.countTasks++;
        this.column[0].push({
          id: this.countTasks,
          task: this.newTask,
          worker: "Alexey",
          dt: new Date(),
          endDt: new Date()
        });
        this.newTask = "";
      }
    },

    moveToNextColomn(columnNumber, elementId) {
      var help = this.column[columnNumber].splice(elementId, 1);
      this.column[columnNumber + 1].push(help[0]);
    },

    moveToColumn(nowColumnNumber, toColumnNumber, elementId) {
      var help = this.column[nowColumnNumber].splice(elementId, 1);
      this.column[toColumnNumber].push(help[0]);
    },

    setEndTime(evt) {
      if (evt.draggedContext.element.task) {
        evt.draggedContext.element.endDt = new Date();
      }
    },
    setTime(evt) {
      if (evt.draggedContext.element.task) {
        evt.draggedContext.element.dt = new Date();
      }
    },
      
    setChosenElement(columnNumber, elementId) {
      this.changed.column = columnNumber;
      this.changed.id = elementId;

      // copy info from modal 
      let element = this.column[columnNumber][elementId];

      this.changed.task = element.task;
      this.changed.worker = element.worker;
      this.changed.dt = element.dt;
      this.changed.endDt = element.endDt;
    },
    //edit save
    saveButton() {
      
      let columnNumber = this.changed.column;
      let elementId = this.changed.id;
      let element = this.column[columnNumber][elementId];

      element.task = this.changed.task;
      element.worker = this.changed.worker;
      element.dt = this.changed.dt;
      element.endDt = this.changed.endDt;

      if (this.changed.status == "Список задач") {
        this.moveToColumn(columnNumber, 0, elementId);
      }
      if (this.changed.status == "В процессе") {
        this.moveToColumn(columnNumber, 1, elementId);
      }
      if (this.changed.status == "Готово") {
        this.moveToColumn(columnNumber, 2, elementId);
      }
    },

    delCard(columnNumber, elementId) {
      this.column[columnNumber].splice(elementId, 1);
    },
    
    toggleStyleLink() {
      var link = document.getElementById("styleLink");

      if (!this.darkChecked) {
        link.href = "./dark.css";
      } else {
        link.href = "";
      }
    }
  }
};
</script>

<style>
.h4 {
  line-height: 1.5;
}
body {
  transition: 0.3s;
  font-family: 'Roboto Mono', monospace !important;
}
.alert {
  border-radius: 0px !important;
}
u {
  text-decoration: none;
  font-weight: bold;
}
.main-colomn {
  border: solid black 2px;
}

.add-text {
  border-radius: 0px!important;
}
.btn {
  border-radius: 0px!important;
  background-color: #23283300!important;
  color: black!important;

}
.but-add {
  background-color: #2F3136!important;
  border-radius: 0px!important;
  color: aliceblue!important;
}
.but-add:hover {
  background-color: #fff!important;
  color: #2F3136!important;
  border-radius: 0px!important;
}
</style>
