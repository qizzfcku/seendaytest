<template>
  <main>
    <fixed-left-column>
      <template v-slot:fixed>
        <Button class="my-element">
          Назад
        </Button>
        <div class="main_div block">
          <div class="block">
            <InputDate
              v-model="selectedDate"
              label="Период"
              :range="true"
              :required="true"
              :with-checkmark="true"
              @blur="handleBlur"
            />
          </div>
          <div class="block">
            <Search
              v-model:searchQuery="searchQuery"
              v-model:searchType="searchType"
              :search-types="searchTypes"
              select-display-value="icon"
              @submit="handleSubmit"
            />
          </div>
          <div class="toggle_block">
            <toggle
              v-for="item in toggleList"
              :key="item.id"
              v-model:pressed="item.isToggled.value"
              class="toggle_block_element"
              color="gray"
              text-orientation="center"
              width="inline"
            >
                <div class="toggle_block_element">{{item.text}}</div>
            </toggle>
          </div>
        </div>
        <div class="main_div block">
          <Select
            v-model="selectedValue"
            label="Год"
            :items="selectItems"
            :editable="true"
          />
          <div class="bttns_element">
            <Button
              size="l"
              @click="ShowButton"
            >
              Показать
            </Button>
            <Button
              size="l"
              color="gray-outline"
              @click="ResetButton"
            >
              Сбросить
            </Button>
          </div>
        </div>
      </template>
      <template v-slot:default>
        <div v-if="!isLoading">
          <StatusCard
            v-for="item in items"
            :key="item.id"
            status="success"
            class="status-card"
          >
            №: {{ item.id }}
          </StatusCard>
        </div>
        <div v-else>
          <Empty
            :without-image="!isLoading"
          >Заказов еще нет!
          </Empty>
        </div>
      </template>
    </fixed-left-column>
  </main>
</template>
<script setup lang="ts">
import { FixedLeftColumn } from "~/shared/ui/templates";
import { InputDate } from "~/shared/ui/inputs/input-date";
import { Search } from "~/shared/ui/search";
import { Toggle } from "~/shared/ui/toggle";
import { Select } from "~/shared/ui/select";
import { StatusCard } from "~/shared/ui/status-card";
import { Empty } from "~/shared/ui/empty";
import {Button } from "~/shared/ui/button";

const selectedDate = ref('');
let firstDate = '';
let secondDate = '';

const searchQuery = ref('');
const searchType = ref('order_number');

const selectedValue = ref('');

const items = ref([]);
const isLoading = ref(false);
const error = ref(null);

const toggleList = [
  { id: 1, text: 'Все', isToggled: ref(false) },
  { id: 2, text: '2', isToggled: ref(false) },
  { id: 3, text: '3', isToggled: ref(false) },
  { id: 4, text: '4', isToggled: ref(false) },
  { id: 5, text: '5', isToggled: ref(false) },
  { id: 6, text: '6', isToggled: ref(false) },
  { id: 7, text: '7', isToggled: ref(false) },
  { id: 8, text: '8', isToggled: ref(false) },
  { id: 9, text: '9', isToggled: ref(false) }
];

const selectItems = [
  { value: '2023' },
  { value: '2022' },
  { value: '2021' }
];


const searchTypes = {
  order_number: { title: 'Номер заказа', placeholder: 'Поиск по номеру заказа' },
  psid: { title: 'Номер фотосессии', placeholder: 'Поиск по номеру фотосессии' },
  client_id: { title: 'Клиент ID', placeholder: 'Поиск по клиенту ID' },
  phone: { title: 'Телефон', placeholder: 'Поиск по телефону' },
  email: { title: 'Email', placeholder: 'Поиск по Email' },
  payer: { title: 'Плательщик, ребенок', placeholder: 'Поиск по плательщику или ребенку' }
};

const handleBlur = () => {
  const [ startDate, endDate ] = selectedDate.value;

  const dateObject1 = new Date(startDate);
  const dateObject2 = new Date(endDate);

  const year1 = dateObject1.getFullYear();
  const month1 = (dateObject1.getMonth() + 1).toString().padStart(2, '0');
  const day1 = dateObject1.getDate().toString().padStart(2, '0');

  const year2 = dateObject2.getFullYear();
  const month2 = (dateObject2.getMonth() + 1).toString().padStart(2, '0');
  const day2 = dateObject2.getDate().toString().padStart(2, '0');

  firstDate = `${year1}${month1}${day1}`;
  secondDate = `${year2}${month2}${day2}`;
  handleSubmit();

};

const handleSubmit = async () => {
  isLoading.value = true;
  try {
    const response = await fetch('http://localhost:3001/lk/method/orders.getTest' + `?search_type=${searchType.value}&search_value=${searchQuery.value}&year=${selectedValue.value}&date_start=${firstDate}&date_finish=${secondDate}`);
    if (!response.ok) {
      throw new Error('Ошибка загрузки данных');
    }
    const data = await response.json();
    items.value = data.response.data.orders;
  } catch (err: any) {
    error.value = err.message;
  } finally {
    if(items.value.length > 0){
      isLoading.value = false;
    }
  }
};

const fetchData = async () => {
  isLoading.value = true;
  try {
  const response = await fetch('http://localhost:3001/lk/method/orders.getTest/');
    if (!response.ok) {
      throw new Error('Ошибка загрузки данных');
    }
    const data = await response.json();
    items.value = data.response.data.orders;
  } catch (err: any) {
    error.value = err.message;
  } finally {
    isLoading.value = false;
  }
};

const ShowButton = () => {
  handleSubmit()
};
const ResetButton = () => {
  selectedValue.value = '';
  fetchData()
};

onMounted(fetchData);


</script>
<style lang="scss" scoped>
@import 'src/shared/assets/styles/general/grid/_mixin.scss';

.main_div{
  background-color: white;
  padding: 15px;
}
.block{
  margin-top:15px;
  margin-bottom:15px;
}
.toggle_block{
  margin-top: 15px;
  margin-bottom: 15px;
}
.toggle_block_element{
  margin: 0 2.5px 5px;
}
.status-card{
  padding: 15px;
}
.bttns_element{
  display: flex;
  justify-content: space-between;
  padding: 16px 0;
}

.my-element {
  width: auto;

  @include start-at(lg) {
    display: none;
  }

  @include active-by(lg) {
    width: 100%;
  }
}
</style>