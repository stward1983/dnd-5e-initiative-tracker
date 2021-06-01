<template>
  <table class="col-12 table table-hover table-sm table-striped">
    <thead>
    <tr>
      <th class="col-2" scope="col"></th>
      <th class="col-6" scope="col">Character</th>
      <th class="col-2 text-center" scope="col">Modifier</th>
      <th class="col-2 text-center" scope="col">Roll</th>
    </tr>
    </thead>
    <tbody>
    <tr :key="character.name" v-for="(character, index) in characters" :class="index % 2 ? 'even' : 'odd'">
      <td class="text-center">
        <a class="bi bi-person-x" @click="removeCharacter(character.uuid)"></a>
      </td>
      <td>{{ character.name }}</td>
      <td class="text-center"><span v-if="character.modifier >= 0">+</span>{{ character.modifier }}</td>
      <td class="text-center">{{ character.roll }}</td>
    </tr>
    <tr :class="characters.length % 2 ? 'even' : 'odd'">
      <td class="text-center">
        <a class="bi bi-person-plus" @click="addCharacter"></a>
      </td>
      <td><input class="form-control form-control-sm" placeholder="Name" type="text" v-model="new_character.name" v-on:keyup.enter="addCharacter"></td>
      <td><input class="form-control form-control-sm text-center" placeholder="Modifier" type="number" v-model="new_character.modifier" v-on:keyup.enter="addCharacter"></td>
      <td></td>
    </tr>
    </tbody>
  </table>

  <button class="bi bi-dice-6 button-primary col-6" @click="rollInitiative"> Roll</button>
  <button class="bi bi-arrow-clockwise button-secondary col-6" @click="resetInitiative"> Reset</button>
</template>

<script>

export default {

  // Set app defaults.
  name: 'D&D 5e Initiative Tracker',
  data() {
    return {
      characters: [],
      new_character: {name: '', modifier: ''}
    }
  },

  // Load character information from local storage.
  mounted() {
    if (localStorage.characters) {
      this.characters = JSON.parse(localStorage.characters);
    }
  },
  methods: {

    // Add a character to the initiative tracker.
    addCharacter() {
      if (!this.new_character.name || !this.new_character.modifier){
        return;
      }
      let character = {
        uuid: this._uuid(),
        name: this.new_character.name,
        modifier: Number(this.new_character.modifier),
        roll: ''
      };
      if (this.characters.length > 0 && this.characters[0].roll) {
        character.roll = Number(character.modifier) + Math.floor(Math.random() * 20);
        this.characters.push(character);
        this.characters.sort((a, b) => (a.roll < b.roll) ? 1 : -1);
      }
      else {
        this.characters.push(character);
        this.characters.sort((a, b) => (a.name > b.name) ? 1 : -1);
      }
      this.new_character = {name: '', modifier: ''};
      localStorage.characters = JSON.stringify(this.characters);
    },

    // Remove a character from the initiative tracker.
    removeCharacter(uuid) {
      this.characters = this.characters.filter((character) => {
        return character.uuid !== uuid
      });
      localStorage.characters = JSON.stringify(this.characters);
    },

    // Remove all rolls and sort the list by character name.
    resetInitiative() {
      let app = this;
      this.characters.forEach((character, index) => {
        app.characters[index].roll = '';
      });
      this.characters.sort((a, b) => (a.name > b.name) ? 1 : -1);
      localStorage.characters = JSON.stringify(this.characters);
    },

    // Roll initiative for all characters in the tracker and put them in order.
    rollInitiative() {
      let app = this;
      this.characters.forEach((character, index) => {
        app.characters[index].roll = character.modifier + Math.floor(Math.random() * 20) + 1;
      });
      this.characters.sort((a, b) => (a.roll < b.roll) ? 1 : -1);
      localStorage.characters = JSON.stringify(this.characters);
    },

    // Generate a UUID.
    // @see https://stackoverflow.com/questions/105034/how-to-create-a-guid-uuid#2117523
    _uuid() {
      return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
        var r = Math.random() * 16 | 0, v = c == 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
      });
    }
  }
}

</script>