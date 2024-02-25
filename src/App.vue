<template>
  <header class="app-header">
    <h1 class="app-title">Карточки</h1>
    <div class="header-info">
      Проект:
      <select v-model="selectedProject" @change="filterCards()">
        <option v-if="selectedSort !== 'pr1' && selectedSort !== 'pr2'" value="">Не выбрано</option>
        <option value="pr1">Проект 1</option>
        <option value="pr2">Проект 2</option>
      </select>
      <button @click="openModal(boardIndex)">Добавить карточку</button>
      <button @click="saveChanges">Сохранить изменения</button>
    </div>
  </header>

  <div class="board-list">
    <div v-for="(board, boardIndex) in boards" :key="board.id" class="board" @dragover.prevent
      @drop="dropItem($event, boardIndex)">
      <div class="board__header">
        <h2 class="board-title">
          <img src="@/assets/ellipse.svg" alt="">
          {{ board.title }}
        </h2>
        <button class="board__filter-button"></button>
      </div>
      <div v-if="board.cards?.length === 0" class="board__empty-message">
        Список карточек
      </div>
      <div v-for="(card, cardIndex) in board.cards" :key="card.id" class="card" draggable="true"
        @dragstart="dragStart($event, boardIndex, cardIndex, 'card')">

        <div class="card__wrapp">
          <div class="card__actions">
            <h3 class="card__title">{{ card.title }}</h3>
            <button class="card__change-button" @click="openEditModal(boardIndex, cardIndex)"><img
                src="@/assets/change.svg" alt=""></button>
            <button class="card__change-button card__delete-button" @click="deleteCard(boardIndex, cardIndex)"><img
                src="@/assets/delete.svg" alt=""></button>
          </div>
          <div class="card__overflow-menu">
            <img src="@/assets/OverflowMenuSecond.png" alt="">
          </div>
        </div>
        <p class="card__score">{{ card.score }}</p>
        <p v-if="card.name" class="card__name">{{ card.name }}</p>
      </div>
      <button class="board__add-button" @click="openModal(boardIndex)">Добавить</button>
    </div>
  </div>


  <div v-if="modalOpen" class="modal">
    <div class="modal__content">
      <button class="modal__close-button" @click="closeModal()"><img src="@/assets/close.png" alt=""></button>
      <h2 class="modal__title">Добавление</h2>
      <select class="modal__select" v-model="selectedProject" @change="filterCards()">
        <option v-if="selectedSort !== 'pr1' && selectedSort !== 'pr2'" value="">Не выбрано</option>
        <option value="st1">Стадия 1</option>
        <option value="st2">Стадия 2</option>
        <option value="st3">Стадия 3</option>
        <option value="st4">Стадия 4</option>
      </select>

      <label class="modal__label" for="cardTitle">Заголовок*:</label>
      <input class="modal__input" id="cardTitle" type="text" v-model="newCard.title" placeholder="Заголовок">
      <label class="modal__label" for="cardProject">Проект:</label>
      <select class="modal__select" v-model="selectedProject" @change="filterCards()">
        <option v-if="selectedSort !== 'pr1' && selectedSort !== 'pr2'" value="">Не выбрано</option>
        <option value="pr1">Проект 1</option>
        <option value="pr2">Проект 2</option>
      </select>
      <label class="modal__label" for="cardScore">Балл*:</label>
      <input class="modal__input" id="cardScore" type="text" v-model="newCard.score" placeholder="Балл">

      <button class="modal__button" @click="addCard(boardIndex)">Добавить</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import data from '@/data.json';

const boards = ref(data.boards);
const modalOpen = ref(false);
const newCard = ref({ title: '', score: '', name: '' });
let boardIndex = ref(null);

const openModal = (index) => {
  modalOpen.value = true;
  newCard.value = { title: '', score: '', name: '' };
  boardIndex.value = index;
};

const closeModal = () => {
  modalOpen.value = false;
};

const addCard = () => {
  const newCardId = Math.max(...boards.value[boardIndex.value].cards.map(card => card.id)) + 1;
  newCard.value.id = newCardId;
  boards.value[boardIndex.value].cards.push(newCard.value);
  closeModal();
  localStorage.setItem('boards', JSON.stringify(boards.value));
};

const openEditModal = (bIndex, cIndex) => {
  showEditModal.value = true;
  editedCard.value = { ...boards.value[bIndex].cards[cIndex] };
  boardIndex.value = bIndex;
  cardIndex.value = cIndex;
};

const deleteCard = (bIndex, cIndex) => {
  boards.value[bIndex].cards.splice(cIndex, 1);
  localStorage.setItem('boards', JSON.stringify(boards.value));
};

const closeEditModal = () => {
  showEditModal.value = false;
};

const saveEditedCard = () => {
  boards.value[boardIndex.value].cards.splice(cardIndex.value, 1, editedCard.value);
  closeEditModal();
  localStorage.setItem('boards', JSON.stringify(boards.value));
};






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

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: start;
}

.modal__label {
  display: block;
  text-align: start;
  font-weight: 400;
  font-size: 12px;
  color: #71747c;
}

.modal__close-button {
  position: absolute;
  right: 10px;
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
}

.modal__select {
  width: 320px;
  background: #f0f3f8;
  margin-bottom: 20px;
}

.modal__content {
  position: relative;
  background-color: #fefefe;
  padding: 30px;
  border: 1px solid #888;
  border-radius: 4px;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

.modal__title {
  font-weight: 700;
  font-size: 20px;
  color: #38393d;
  margin-bottom: 12px;
}

.modal__input {
  width: 320px;
  display: block;
  border-radius: 4px;
  padding: 8px;
  height: 32px;
  background: #f0f3f8;
  font-weight: 400;
  font-size: 13px;
  color: #000;
  outline: none;
  border: none;
  margin-bottom: 20px;
}

.app-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
}

.header-info {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #7d8ca1;
}


select {
  border-radius: 4px;
  padding: 8px;
  height: 32px;
  background: #d5dce5;
  font-weight: 400;
  font-size: 13px;
  color: #7d8ca1;
  outline: none;
  border: none;
}

.board-list {
  display: flex;
  justify-content: space-between;
  gap: 20px;
}

.board__filter-button {
  background: url("../src/assets/sort.png");
}

.board-title {
  align-items: center;
  display: flex;
  gap: 7px;
  font-weight: 600;
  font-size: 15px;
  color: #38393d;
  text-align: start;
}

.board {
  border-radius: 4px;
  padding: 12px;
  width: 320px;
  height: max-content;
  background-color: #D5DCE5;
}

.board__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  align-items: center;
  text-align: center;
  margin-bottom: 12px;
}

.card {

  border-radius: 4px;
  padding: 12px;
  margin: 0;
  cursor: move;
  background-color: #fff;
  text-align: start;
  margin-bottom: 8px;
}

.card__wrapp {
  display: flex;
  justify-content: space-between;
}

.card__actions {
  display: flex;
  gap: 8px;
  align-items: center;
  margin-bottom: 8px;

}

.card__title {
  font-weight: 600;
  font-size: 14px;
  color: #38393d;
}

.card__name {
  display: flex;
  justify-content: center;
  align-items: center;
  background: #e5e9ef;
  border: 1px solid #d2dae4;
  border-radius: 4px;
  width: 70px;
  height: 23px;
  font-weight: 400;
  font-size: 12px;
  color: #7d8ca1;
}

.card__score {
  font-weight: 400;
  font-size: 12px;
  color: #71747c;
  margin-bottom: 33px;
}

.board__add-button {
  background-color: inherit;
  color: #7d8ca1;
}

.card__change-button {
  display: flex;
  background-color: inherit;
  padding: 0;
}

.card__change-button img {
  width: 17px;
  height: 17px;
}

@media (max-width: 768px) {
  .app-header {
    flex-direction: column;
    align-items: center;
  }

  .header-info {
    margin-top: 10px;
    flex-direction: column;
    align-items: normal;
  }

  .board-list {
    flex-direction: column;
  }

  .board {
    width: 100%;
    margin-top: 20px;
  }

}
</style>