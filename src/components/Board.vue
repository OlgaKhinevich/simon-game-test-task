<template>
  <div>
    <div class="game">
        <header>
            <h1 class="heading">Simon Game</h1>
            <h3 v-if="!this.level == 0">Уровень: {{this.level}} из 20</h3>
        </header>

        <section 
            :class=" clickability ? 'buttons-container' : 'buttons-container unclickable'">
            <div 
                :class="button == 'red' ? 'button button-red activated' : 'button button-red'"
                @click="handleClick('red')"></div>
            <div
                :class="button == 'green' ? 'button button-green activated' : 'button button-green'"
                @click="handleClick('green')"
            ></div>
            <div 
                :class="button == 'blue' ? 'button button-blue activated' : 'button button-blue'"
                @click="handleClick('blue')"
            ></div>
            <div
                :class="button == 'yellow' ? 'button button-yellow activated' : 'button button-yellow'"
                @click="handleClick('yellow')" 
            ></div>
        </section>

        <footer class="info-section">
            <p class="note">{{note}}</p>
            <div v-if="this.level == 0">
                <p>Выберите уровень скорости:</p><br>
                <p v-for="(speed, index) in speedLevel" :key="index"><input 
                    class="radio-btn"
                    name="speed"
                    type="radio"
                    :value="speed"
                    :id="index"
                    v-model="selectedSpeed">
                    <label :for="index"> {{speed.name}}</label>
                </p>
            </div>
            <button v-if="this.level == 0" class="start-button" @click="startGame">Старт</button>
        </footer>
    </div>
  </div>
</template>

<script>
export default {
    data() {
        return {
            sequence: [],
            humanSequence: [],
            level: 0,
            nextSequence: [],
            buttons: ['red', 'green', 'blue', 'yellow'],
            button: null,
            sound: null,
            selectedSpeed: {
                name: 'Легкий',
                value: 1500
            },
            speedLevel: [
                {
                    name: 'Легкий',
                    value: 1500
                },
                {
                    name: 'Средний',
                    value: 1000
                },
                {
                    name: 'Сложный',
                    value: 400
                },
            ],
            clickability: false,
            note: '',
            remainingTaps: 0
        }
    },
    mounted() {
        this.button = document.getElementById('app').dataset.button;
        this.sound = document.getElementById('app').dataset.sound;
    },
    computed: {
    },
    methods: {
        startGame() {
            this.startNextRound();
        },
        chooseSound(value) {
            switch(value) {
                case 'red':
                this.sound = 'https://s3.amazonaws.com/freecodecamp/simonSound1.mp3';
                break;
                case 'green':
                this.sound = 'https://s3.amazonaws.com/freecodecamp/simonSound2.mp3';
                break;
                case 'blue':
                this.sound = 'https://s3.amazonaws.com/freecodecamp/simonSound3.mp3';
                break;
                case 'yellow':
                this.sound = 'https://s3.amazonaws.com/freecodecamp/simonSound4.mp3';
                break;
            }
            return this.sound;
        },
        nextStep() {
            return this.buttons[Math.floor(Math.random() * this.buttons.length)];
        },
        startNextRound() {
            this.note = "Ожидание хода компьютера...";
            this.level++;
            this.clickability = false;
            this.nextSequence = [...this.sequence];
            this.nextSequence.push(this.nextStep());
            this.playRound();
            this.sequence = [...this.nextSequence];
            setTimeout(() => {
                this.humanTurn();
            }, this.level * this.selectedSpeed.value + 1000);
        },
        activateButton(color) {
            this.button = color;
            setTimeout(() => {
                this.button = '';
            }, 300);
        },
        playRound() {
            this.nextSequence.forEach((color, index) => {
            setTimeout(() => {
                this.activateButton(color);
                }, (index + 1) * this.selectedSpeed.value);
            });
        },
        humanTurn() {
            this.clickability = true;
            this.note = 'Ваш ход';
        },
        handleClick(value) {
            this.chooseSound(value);
            const index = this.humanSequence.push(value) - 1;
            const audio = new Audio(this.sound);
            audio.play();

            this.remainingTaps = this.sequence.length - this.humanSequence.length;
            if (this.humanSequence[index] !== this.sequence[index]) {
                this.note = 'Вы ошиблись! Игра окончена!';
                this.resetGame();
                return;
            }

            if (this.humanSequence.length === this.sequence.length) {
                if (this.humanSequence.length === 20) {
                    this.note = 'Поздравляем! Вы прошли все уровни!';
                    this.resetGame();
                    return;
                }
                this.note = 'Правильно! Вы перешли на следующий уровень!';
                this.humanSequence = [];
                setTimeout(() => {
                    this.startNextRound();
                }, 1000);
                return;
            }
            if(this.remainingTaps > 0) this.note = `Нажмите еще ${this.remainingTaps} кликов`;
        },
        resetGame() {
            this.sequence = [];
            this.humanSequence = [];
            this.level = 0;
            this.clickability = false;
        }
    }
}
</script>

<style>
    html { box-sizing: border-box; }

    *,
    *::after,
    *::before {
        margin: 0;
        padding: 0;
        box-sizing: inherit;
    }

    body {
        display: flex;
        min-width: 100vw;
        min-height: 100vh;
        overflow-x: hidden;
        justify-content: center;
    }

    .game {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        margin-bottom: 10px 0 15px 0;
    }

    .hidden { display: none !important; }

    .invisble { visibility: hidden; }

    header { font-size: 18px; }

    .heading {
        font-size: 2em;
        margin-top: 20px;
    }

    .buttons-container {
        display: grid;
        justify-content: center;
        grid-template-rows: 160px 160px;
        grid-template-columns: 160px 160px;
        grid-gap: 20px;
        position: relative;
        margin: 20px 0 30px 0;
    }

    .unclickable { pointer-events: none; }

    .button-red {
        background-color: #E53935;
        box-shadow: 0 0 0 1px #E53935 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 8px 0 0 #D32F2F, 0 8px 0 1px rgba(0, 0, 0, 0.4), 0 8px 8px 1px rgba(0, 0, 0, 0.5);
        transition: 0.15s;
        border-radius: 100% 0 0 0;
    }

    .button-red:hover,
    .button-red:focus {
        background-color: #F44336;
        box-shadow: 0 0 0 1px #F44336 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 10px 0 0 #D32F2F, 0 10px 0 1px rgba(0, 0, 0, 0.4), 0 10px 8px 1px rgba(0, 0, 0, 0.6);
    }

    .button-red:active,
    .button-red.activated {
        box-shadow: 0 0 0 1px #FF1744 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 0 0 1px rgba(0, 0, 0, 0.4);
        background-color: #FF1744;
        transform: translateY(10px);
    }

    .button-green {
        background-color: #43A047;
        box-shadow: 0 0 0 1px #43A047 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 8px 0 0 #388E3C, 0 8px 0 1px rgba(0, 0, 0, 0.4), 0 8px 8px 1px rgba(0, 0, 0, 0.5);
        transition: 0.15s;
        border-radius: 0 100% 0 0;
    }

    .button-green:hover,
    .button-green:focus {
        background-color: #4CAF50;
        box-shadow: 0 0 0 1px #4CAF50 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 10px 0 0 #388E3C, 0 10px 0 1px rgba(0, 0, 0, 0.4), 0 10px 8px 1px rgba(0, 0, 0, 0.6)
    }

    .button-green:active,
    .button-green.activated {
        box-shadow: 0 0 0 1px #00E676 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 0 0 1px rgba(0, 0, 0, 0.4);
        background-color: #00E676;
        transform: translateY(10px);
    }

    .button-yellow {
        background-color: #FFB300;
        box-shadow: 0 0 0 1px #FFB300 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 8px 0 0 #FFA000, 0 8px 0 1px rgba(0, 0, 0, 0.4), 0 8px 8px 1px rgba(0, 0, 0, 0.5);
        transition: 0.15s;
        border-radius: 0 0 100% 0;
    }

    .button-yellow:hover,
    .button-yellow:focus {
        background-color: #FFC107;
        box-shadow: 0 0 0 1px #FFC107 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 8px 0 0 #FFA000, 0 8px 0 1px rgba(0, 0, 0, 0.4), 0 8px 8px 1px rgba(0, 0, 0, 0.5);
    }

    .button-yellow:active,
    .button-yellow.activated {
        box-shadow: 0 0 0 1px #FFC400 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 0 0 1px rgba(0, 0, 0, 0.4);
        background-color: #FFC400;
        transform: translateY(10px);
    }

    .button-blue {
        background-color: #039BE5;
        box-shadow: 0 0 0 1px #039BE5 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 8px 0 0 #0288D1, 0 8px 0 1px rgba(0, 0, 0, 0.4), 0 8px 8px 1px rgba(0, 0, 0, 0.5);
        transition: 0.15s;
        border-radius: 0 0 0 100%;
    }

    .button-blue:hover,
    .button-blue:focus {
        background-color: #03A9F4;
        box-shadow: 0 0 0 1px #03A9F4 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 10px 0 0 #0288D1, 0 10px 0 1px rgba(0, 0, 0, 0.4), 0 10px 8px 1px rgba(0, 0, 0, 0.6);
    }

    .button-blue:active,
    .button-blue.activated {
        box-shadow: 0 0 0 1px #00B0FF inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset, 0 0 0 1px rgba(0, 0, 0, 0.4);
        background-color: #00B0FF;
        transform: translateY(10px);
    }

    .info-section {
        text-align: center;
        font-size: 20px;
    }

    .info-section .note {
        font-weight: 700;
        font-size: 24px;
        margin-bottom: 20px;
    }

    .info-section p { margin-bottom: 5px; }

    .start-button {
        display: inline-block;
        height: 38px;
        padding: 0 30px;
        color: rgb(31, 31, 31);
        text-align: center;
        font-size: 18px;
        font-weight: 600;
        line-height: 38px;
        letter-spacing: 0.1rem;
        text-transform: uppercase;
        text-decoration: none;
        white-space: nowrap;
        background-color: transparent;
        border-radius: 5px;
        border: 1px solid #bbb;
        cursor: pointer;
        margin: 10px 0 30px 0;
    }

    .start-button:hover {
        color: #333;
        border-color: #888;
        outline: 0;
    }

    .info { font-size: 22px; }

    @media screen and (max-width: 550px) {
        .game {
            padding-left: 20px;
            padding-right: 20px;
        }
        .button-container {
            grid-template-rows: 100px 100px;
            grid-template-columns: 100px 100px;
        }
    }
</style>