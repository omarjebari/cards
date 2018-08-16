<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card card-default">
                    <div class="card-header">Play Your Cards Right</div>
                    <div class="card-body">
                        Choose whether the next card is higher or lower than the current one
                    </div>
                    <div v-if="cards.length" class="card-body">
                        <div>
                            {{ cards[currentIndex].value }} {{ cards[currentIndex].suit }}
                        </div>
                        <div>
                            <button type="button" class="btn btn-primary" @click="checkChoice('higher')" :disabled="gameOver">Higher</button>
                            <button type="button" class="btn btn-primary" @click="checkChoice('lower')" :disabled="gameOver">Lower</button>
                        </div>
                        <div v-if="gameOver">
                            <div>Score: {{ countCorrect }}</div>
                            <button type="button" class="btn btn-primary" @click="newGame()">New game</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    /**
     * A Play Your Cards Right component
     * - The cards are pulled from a remote source and shuffled
     * - The user keeps playing until they get 3 wrong
     * - If the user gets to the end of the deck the cards are reshuffled and the user continues
     * - When the game is over the user's total is displayed with the option of starting a new game
     */
    export default {
        data: function() {
            return {
                cards: [],
                lives: 3,
                countCorrect: 0,
                countTotal: 0,
                currentIndex: 0,
                cardValueMap: {
                    'A': 1,
                    'K': 13,
                    'Q': 12,
                    'J': 11
                },
            }
        },
        computed: {
            gameOver() {
                return this.lives === 0
            }
        },
        filters: {},
        methods: {
            getData() {
                let self = this;
                axios.get('/get-data')
                    .then(function (response) {
                        self.cards = response.data;
                        self.shuffle();
                    })
                    .catch(function (error) {
                        // handle error
                        console.log(error);
                    })
            },
            shuffle() {
                this.cards = _.shuffle(this.cards);
            },
            /**
             * Compare the current card with the next one and update the score.
             * If got to the end of the deck then reshuffle and continue.
             * @param choice
             */
            checkChoice(choice) {
                let currentCardValue = this.mapCardValue(this.cards[this.currentIndex].value);
                let nextCardValue = this.mapCardValue(this.cards[this.currentIndex + 1].value);
                if (choice === 'higher') {
                    if (nextCardValue >= currentCardValue) {
                        this.setCorrect();
                    }
                    else {
                        this.setIncorrect();
                    }
                }
                else {
                    if (nextCardValue < currentCardValue) {
                        this.setCorrect();
                    }
                    else {
                        this.setIncorrect();
                    }
                }
                if (this.currentIndex === 50) {     // Got to the end of the deck so shuffle and start again
                    this.shuffle();
                    this.currentIndex = 0;
                }
                else {
                    ++this.currentIndex;
                    ++this.countTotal;
                }
            },
            setCorrect() {
                ++this.countCorrect;
            },
            setIncorrect() {
                --this.lives;
            },
            newGame() {
                this.currentIndex = 0;
                this.lives = 3;
                this.countCorrect = 0;
                this.countTotal = 0;
                this.shuffle();
            },
            mapCardValue(value) {
                return isNaN(value) ? this.cardValueMap[value] : value;
            }
        },
        mounted() {
            this.getData();
        },
        created() {}
    }
</script>
