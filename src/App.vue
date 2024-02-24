<template>
  <div id="app" class="app-container">
    <header class="app-header">
      <h1 class="app-title">Карточки</h1>
      <div class="header-info">
    Проект:
    <select v-model="selectedProject" @change="filterCards()">
      <option v-if="selectedSort !== 'pr1' && selectedSort !== 'pr2'" value="">не выбрано</option>
      <option value="pr1">Проект 1</option>
      <option value="pr2">Проект 2</option>
    </select>
    <button @click="showModal = true">Добавить карточку</button>
    <button @click="saveChanges">Сохранить изменения</button>
  </div>
    </header>

    <div class="board-list">
      <div v-for="(board, boardIndex) in boards" :key="board.id" class="board" @dragover.prevent
        @drop="dropItem($event, boardIndex)">
        <h2 class="board-title">{{ board.title }}</h2>
        <div v-if="board.cards?.length === 0" class="empty-board-message">
          Список карточек пуст
        </div>
        <div v-for="(card, cardIndex) in board.cards" :key="card.id" class="card" draggable="true"
          @dragstart="dragStart($event, boardIndex, cardIndex, 'card')">
          <h3 class="card-title">{{ card.title }}</h3>
          <p class="card-score">{{ card.score }}</p>
          <p v-if="card.name" class="card-name">{{ card.name }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import data from '@/data.json';  

const boards = ref(data.boards);  


const selectedProject = ref('');

const filterCards = () => {
  boards.value.forEach(board => {
    if (selectedProject.value === '') { 
      board.cards = board.cards;
    } else { 
      board.cards = board.cards.filter(card => card.name === selectedProject.value);
    }
  });
};

const dragStart = (event, boardIndex, itemIndex, type) => {
  event.dataTransfer.setData('boardIndex', boardIndex);
  event.dataTransfer.setData('itemIndex', itemIndex);
  event.dataTransfer.setData('type', type);
};

const dropItem = (event, targetBoardIndex) => {
  const sourceBoardIndex = event.dataTransfer.getData('boardIndex');
  const itemIndex = event.dataTransfer.getData('itemIndex');
  const type = event.dataTransfer.getData('type');

  if (sourceBoardIndex === targetBoardIndex) {
    return;
  }

  const item = boards.value[sourceBoardIndex][type + 's'].splice(itemIndex, 1)[0];
  boards.value[targetBoardIndex][type + 's'].push(item);

  localStorage.setItem('boards', JSON.stringify(boards.value));
};
</script>

<style>
.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: #f8f8f8;
  border-bottom: 1px solid #ddd;
}

.header-info {
  display: flex;
  align-items: center;
  gap: 10px;
  /* Расстояние между элементами внутри .header-info */
}

/* Добавьте этот стиль, если хотите, чтобы кнопки были в один ряд */
.header-info button {
  margin-left: 10px;
  /* Расстояние между кнопками */
}

.board-list {
  display: flex;
  justify-content: space-between;
}

.board {
  border: 1px solid #000;
  padding: 10px;
  margin: 10px;
  width: 250px;
  min-height: 300px;
  background-color: #D5DCE5;
}

.card {
  border: 1px solid #ccc;
  padding: 10px;
  margin: 10px;
  cursor: move;
  background-color: #fff;
  text-align: start;
}

.card-title {
  font-weight: 600;
  font-size: 14px;
  color: #38393d;
}

.card-name {
  background: #e5e9ef;
  border: 1px solid #d2dae4;
  border-radius: 4px;
  padding: 4px 8px;
  width: 68px;
  height: 23px;
}</style>