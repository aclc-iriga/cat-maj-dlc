<template>
    <v-table
        v-if="eventSlug && event"
        density="comfortable"
        fixed-header
        :height="scoreSheetHeight"
        :id="componentId"
    >
        <thead>
            <tr>
                <th
                    colspan="3"
                    class="text-uppercase text-center font-weight-bold text-grey-darken-4 py-3"
                    :class="$vuetify.display.mdAndDown ? 'text-h5' : 'text-h4'"
                >
                    {{ event.title }}
                </th>
                <th
                    v-for="(criterion, criterionIndex) in criteria"
                    style="width: 13%"
                    class="text-center text-uppercase py-3"
                    :class="{ 'bg-grey-lighten-4': coordinates.x == criterionIndex && !scoreSheetDisabled }"
                >
                    <div class="d-flex h-100 flex-column align-content-space-between">
                        <p
                            class="text-grey-darken-1"
                            :class="{
                                'text-caption text-uppercase': $vuetify.display.mdAndDown,
                                'text-grey-darken-3': coordinates.x == criterionIndex && !scoreSheetDisabled
                            }"
                        >
                            {{ criterion.title }}
                        </p>
                        <b
                            class="text-grey-darken-2 text-h6 pt-1"
                            :class="{
                                'text-body-2 text-uppercase font-weight-bold': $vuetify.display.mdAndDown,
                                'text-grey-darken-4': coordinates.x == criterionIndex && !scoreSheetDisabled
                            }"
                            style="margin-top: auto"
                        >
                            {{ criterion.percentage }}%
                        </b>
                    </div>
                </th>
                <th
                    style="width: 13%"
                    class="text-uppercase text-center text-grey-darken-3 font-weight-bold py-3"
                    :class="{ 'bg-grey-lighten-4': coordinates.x == criteria.length && !scoreSheetDisabled }, $vuetify.display.mdAndDown ? 'text-body-1' : 'text-h6'"
                >
                    Total
                    <p class="ma-0 text-subtitle-2 text-green-darken-2">{{ minRating.toFixed(2) }} - {{ (maxRating >= 100) ? maxRating.toFixed(0) : maxRating.toFixed(2) }}</p>
                </th>
                <th
                    style="width: 13%"
                    class="text-uppercase text-center text-grey-darken-3 font-weight-bold py-3"
                    :class="$vuetify.display.mdAndDown ? 'text-body-1' : 'text-h6'"
                >
                    Rank
                    <p class="ma-0 text-subtitle-2">&nbsp;</p>
                </th>
            </tr>
        </thead>
        <tbody>
            <tr
                v-for="(team, teamIndex) in teams"
                :key="team.id"
                :class="{ 'bg-grey-lighten-4': coordinates.y == teamIndex && !scoreSheetDisabled }"
            >
                <td
                    class="text-uppercase text-right text-h4 font-weight-bold text-grey-darken-2"
                    :class="{ 'text-grey-darken-4': coordinates.y == teamIndex && !scoreSheetDisabled }"
                >
                    {{ team.number }}
                </td>
                <td style="width: 72px;">
                    <v-avatar size="72">
                        <v-img
                            cover
                            :src="`${$store.getters.appURL}/crud/uploads/${team.avatar}`"
                        />
                    </v-avatar>
                </td>
                <td
                    class="px-0 text-grey-darken-2"
                    :class="{ 'text-grey-darken-4': coordinates.y == teamIndex && !scoreSheetDisabled }"
                >
                    <p class="ma-0 text-subtitle-2 text-uppercase font-weight-bold" style="line-height: 1.2">{{ team.name }}</p>
                    <p class="mt-1 mb-0" style="line-height: 1"><small>{{ team.location }}</small></p>
                </td>
                <td
                    v-for="(criterion, criterionIndex) in criteria"
                    :key="criterion.id"
                    :class="{ 'bg-grey-lighten-4': coordinates.x == criterionIndex && !scoreSheetDisabled }"
                >
                    <v-text-field
                        type="number"
                        class="font-weight-bold"
                        hide-details
                        single-line
                        :min="0"
                        :max="criterion.percentage"
                        @change="saveRating(ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`], criterion.percentage, team)"
                        v-model.number="ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value"
                        :class="{
                            'text-error font-weight-bold': (
                                ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value < 0 ||
                                ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value > criterion.percentage
                            ),
                            'text-grey-darken-1': ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value === 0,
                            'text-grey-darken-3': (
                                ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value > 0 &&
                                ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value <= criterion.percentage
                            )
                        }"
                        :error="(
                              ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value.toString().trim() === ''
                           || ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value < 0
                           || ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value > criterion.percentage
                        )"
                        :variant="
                            ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value < 0
                            ||
                            ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value > criterion.percentage
                            ? 'outlined' : 'underlined'
                        "
                        :disabled="team.disabled || ratings[`${event.slug}_${team.id}`][`${$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].is_locked"
                        :id="`input_${teamIndex}_${criterionIndex}`"
                        @keyup.prevent="handleRatingKeyUp(team)"
                        @keydown.down.prevent="moveDown(criterionIndex, teamIndex)"
                        @keydown.enter="moveDown(criterionIndex, teamIndex)"
                        @keydown.up.prevent="moveUp(criterionIndex, teamIndex)"
                        @keydown.right.prevent="moveRight(criterionIndex, teamIndex)"
                        @keydown.left.prevent="moveLeft(criterionIndex, teamIndex)"
                        @focus.passive="updateCoordinates(criterionIndex, teamIndex)"
                    />
                </td>
                <td :class="{ 'bg-grey-lighten-4': coordinates.x == criteria.length && !scoreSheetDisabled }">
                    <v-text-field
                        type="number"
                        class="font-weight-bold"
                        variant="outlined"
                        hide-details
                        single-line
                        :loading="totals[`team_${team.id}`].loading"
                        v-model.number="totals[`team_${team.id}`].value"
                        :min="minRating"
                        :max="maxRating"
                        @change="calculateTotalScores(team)"
                        :class="{
                            'text-error font-weight-bold': (
                                totals[`team_${team.id}`].value < minRating
                            || totals[`team_${team.id}`].value > maxRating
                            ),
                            'text-green-darken-2 font-weight-bold': (
                                totals[`team_${team.id}`].value >= minRating
                            && totals[`team_${team.id}`].value <= maxRating
                            )
                        }"
                        :error="(
                              totals[`team_${team.id}`].value.toString().trim() === ''
                           || totals[`team_${team.id}`].value < minRating
                           || totals[`team_${team.id}`].value > maxRating
                        )"
                        :disabled="team.disabled || totals[`team_${team.id}`].is_locked"
                        :id="`input_${teamIndex}_${criteria.length}`"
                        @keydown.down.prevent="moveDown(criteria.length, teamIndex)"
                        @keydown.enter="moveDown(criteria.length, teamIndex)"
                        @keydown.up.prevent="moveUp(criteria.length, teamIndex)"
                        @keydown.right.prevent="moveRight(criteria.length, teamIndex)"
                        @keydown.left.prevent="moveLeft(criteria.length, teamIndex)"
                        @focus.passive="updateCoordinates(criteria.length, teamIndex)"
                    />
                </td>
                <td
                    class="text-center text-h6"
                    :class="{
                        'text-grey-darken-2': coordinates.y != teamIndex && !scoreSheetDisabled,
                        'text-grey-darken-4': coordinates.y == teamIndex && !scoreSheetDisabled,
                        'text-grey-darken-1': scoreSheetDisabled,
                    }"
                >
                    <span :style="{'opacity': team.disabled ? 0.6 : 1}">{{ ranks[`team_${team.id}`] }}</span>
                </td>
            </tr>
        </tbody>
        <tfoot>
            <tr>
                <td colspan="12">
                    <v-col align="center"
                           justify="end"
                    >
                        <v-btn
                            class="py-7 bg-grey-lighten-1 text-grey-darken-3"
                            @click="openSubmitDialog"
                            :disabled="scoreSheetDisabled"
                            block
                            flat
                        >
                        <p style="font-size: 1.2rem;">submit ratings</p>
                        </v-btn>
                        <v-dialog
                            v-if="submitDialog"
                            v-model="submitDialog"
                            persistent
                            max-width="400"
                        >
                            <v-card>
                                <v-card-title class="bg-black">
                                    <v-icon id="remind">mdi-information</v-icon> Submit Ratings
                                </v-card-title>
                                <v-card-text>
                                    Please confirm that you wish to finalize the ratings for <b>{{ event.title }}</b>. This action cannot be undone.
                                </v-card-text>
                                <v-card-actions>
                                    <v-spacer></v-spacer>
                                    <v-btn
                                        class="text-red-darken-1"
                                        :disabled="submitLoading"
                                        @click="submitDialog = false"
                                    >
                                        Go Back
                                    </v-btn>
                                    <v-btn
                                        class="text-green-darken-1"
                                        :loading="submitLoading"
                                        @click="submitRatings"
                                    >
                                        Submit
                                    </v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>
                        <v-dialog
                            v-if="inspectDialog"
                            v-model="inspectDialog"
                            persistent
                            max-width="400"
                        >
                            <v-card>
                                <v-card-title class="bg-red-darken-4">
                                    <v-icon id="warning">mdi-alert</v-icon>	Submit Ratings
                                </v-card-title>
                                <v-card-text>
                                    <p class="mb-2 text-red-darken-4">
                                        Sorry, your ratings for <b>{{ event.title }}</b> cannot be submitted as they must be from
                                        <big><b>{{ minRating }}</b></big> to <big><b>{{ maxRating }}</b></big>.
                                    </p>
                                    <p class="text-red-darken-4">Please adjust your ratings and try submitting again.</p>
                                </v-card-text>
                                <v-card-actions>
                                    <v-spacer></v-spacer>
                                    <v-btn color="red-darken-4" @click="inspectDialog = false">Go Back</v-btn>
                                </v-card-actions>
                            </v-card>
                        </v-dialog>
                    </v-col>
                </td>
            </tr>
        </tfoot>
    </v-table>

    <!-- loader -->
    <div v-else-if="this.eventSlug" class="d-flex justify-center align-center" style="height: 100vh;">
        <v-progress-circular
            :size="80"
            color="black"
            class="mb-16"
            indeterminate
        />
    </div>
</template>


<script>
    import $ from 'jquery';

    export default {
        name: 'JudgeRating',
        props: ['eventSlug'],
        data() {
            return {
                dialog: false,
                submitDialog: false,
                submitLoading: false,
                inspectDialog: false,
                event: null,
                teams: [],
                criteria: [],
                ratings: {},
                totals: {},
                coordinates: {
                    x: -1,
                    y: -1
                }
            }
        },
        computed: {
            componentId() {
                let id = 'rating';
                if(this.eventSlug)
                    id += `__${this.eventSlug}`;
                return id;
            },
            ranks() {
                const teamRanks = {};

                // get unique totals
                const uniqueTotals = [];
                for (let i = 0; i < this.teams.length; i++) {
                    const team = this.teams[i];
                    const teamKey = `team_${team.id}`;
                    const total = this.totals[teamKey].value;
                    if (!uniqueTotals.includes(total))
                        uniqueTotals.push(total);

                    // push to teamRanks
                    teamRanks[teamKey] = 0;
                }

                // sort uniqueTotals in descending order
                uniqueTotals.sort((a, b) => b - a);

                // prepare rankGroup
                const rankGroup = {};

                // get dense rank of each team
                const denseRanks = {};
                for (let i = 0; i < this.teams.length; i++) {
                    const team = this.teams[i];
                    const teamKey = `team_${team.id}`;
                    const total = this.totals[teamKey].value;
                    const denseRank = 1 + uniqueTotals.indexOf(total);
                    denseRanks[denseRank] = denseRank;

                    // push to rankGroup
                    const rankGroupKey = `rank_${denseRank}`;
                    if (!rankGroup[rankGroupKey])
                        rankGroup[rankGroupKey] = [];
                    rankGroup[rankGroupKey].push(teamKey);
                }

                // get fractional rank
                let ctr = 0;
                for (let i = 0; i < uniqueTotals.length; i++) {
                    const key = `rank_${(i + 1)}`;
                    const group = rankGroup[key];
                    const size = group.length;
                    const fractionalRank = ctr + (((size * (size + 1)) / 2) / size);

                    // write fractionalRank to group members
                    for (let j = 0; j < size; j++) {
                        teamRanks[group[j]] = fractionalRank;
                    }
                    ctr += size;
                }
                return teamRanks;
            },
            scoreSheetHeight() {
                return this.$store.getters.windowHeight - 64;
            },
            scoreSheetDisabled() {
                // initialize disable
                let disabled = true;
                // get ratings.is_locked and pass value to disabled variable
                for (let i = 0; i < this.teams.length; i++) {
                    const ratings = this.ratings[`${this.event.slug}_${this.teams[i].id}`];
                    for (let j = 0; j < this.criteria.length; j++) {
                        const criterion = this.criteria[j];
                        const rating = ratings[`${this.$store.getters['auth/getUser'].id}_${criterion.id}_${this.teams[i].id}`]
                        if (!rating.is_locked) {
                            disabled = false;
                            break;
                        }
                    }
                }
                // return value
                return disabled;
            },
            totalCriteriaPercentage() {
                let percentage = 0;
                for(let i=0; i<this.criteria.length; i++) {
                    percentage += this.criteria[i].percentage;
                }
                return percentage;
            },
            minRating() {
                if(this.totalCriteriaPercentage >= 100)
                    return this.$store.state.rating.min;
                else
                    return this.totalCriteriaPercentage * 0.70;
            },
            maxRating() {
                if(this.totalCriteriaPercentage >= 100)
                    return this.$store.state.rating.max;
                else
                    return this.totalCriteriaPercentage;
            }
        },
        methods: {
            fetchScoreSheet() {
                // fetch scoreSheet from backend
                if (this.eventSlug) {
                    $.ajax({
                        url: `${this.$store.getters.appURL}/${this.$store.getters['auth/getUser'].userType}.php`,
                        type: 'GET',
                        xhrFields: {
                            withCredentials: true
                        },
                        data: {
                            getScoreSheet: this.eventSlug,
                            duoSlug: this.$route.params.duoSlug ? this.$route.params.duoSlug : null
                        },
                        success: (data) => {
                            data = JSON.parse(data);
                            this.criteria = data.criteria;
                            this.teams = data.teams;
                            this.ratings = data.ratings;
                            this.event = data.event;
                            this.totals = {}
                            // create total score for ratings
                            for (let i = 0; i < this.teams.length; i++) {
                                let total = 0;
                                this.totals[`team_${this.teams[i].id}`] = {};
                                const rating = this.ratings[`${this.event.slug}_${this.teams[i].id}`];
                                for (let j = 0; j < this.criteria.length; j++) {
                                    const criterion = this.criteria[j];
                                    const value = rating[`${this.$store.getters['auth/getUser'].id}_${criterion.id}_${this.teams[i].id}`].value
                                    this.totals[`team_${this.teams[i].id}`].is_locked = rating[`${this.$store.getters['auth/getUser'].id}_${criterion.id}_${this.teams[i].id}`].is_locked
                                    total += value;
                                }
                                this.totals[`team_${this.teams[i].id}`].value = total;
                                this.totals[`team_${this.teams[i].id}`].loading = false;
                            }
                        },
                        error: (error) => {
                            alert(`ERROR ${error.status}: ${error.statusText}`);
                        },
                    });
                }
            },
            handleRatingKeyUp(team) {
                // initialize total
                let total = 0;

                // accumulate ratings to total score
                const teamRating = this.ratings[`${this.event.slug}_${team.id}`];
                for (let i = 0; i < this.criteria.length; i++) {
                    const criterion = this.criteria[i];
                    total += Number(teamRating[`${this.$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`].value);
                }

                // accumulate total adds into totals object
                this.totals[`team_${team.id}`].value = total;
            },
            saveRating(rating, percentage, team) {
                this.totals[`team_${team.id}`].loading = true;

                // validate score
                if (rating.value < 0 || rating.value === '')
                    rating.value = 0;
                else if (rating.value > percentage)
                    rating.value = percentage;
                this.handleRatingKeyUp(team);

                // auto-save ratings
                $.ajax({
                    url: `${this.$store.getters.appURL}/${this.$store.getters['auth/getUser'].userType}.php`,
                    type: 'POST',
                    xhrFields: {
                        withCredentials: true
                    },
                    data: {
                        rating
                    },
                    success: (data, textStatus, jqXHR) => {
                        // set timeout for loading
                        if (this.totals[`team_${team.id}`].loading) {
                            setTimeout(() => {
                                this.totals[`team_${team.id}`].loading = false;
                            }, 1000);
                        }
                    },
                    error: (error) => {
                        alert(`ERROR ${error.status}: ${error.statusText}`);
                    }
                });
            },
            calculateTotalScores(team) {
                // set loading state
                this.totals[`team_${team.id}`].loading = true;

                // validates total scores
                if (this.totals[`team_${team.id}`].value < this.minRating || this.totals[`team_${team.id}`].value === '') {
                    this.totals[`team_${team.id}`].value = this.minRating;
                } else if (this.totals[`team_${team.id}`].value > this.maxRating) {
                    this.totals[`team_${team.id}`].value = this.maxRating;
                }

                // total score divided and distributed based on criteria percentage
                let ratings = [];
                for (let criterion of this.criteria) {
                    const rating = this.ratings[`${this.event.slug}_${team.id}`][`${this.$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`];
                    rating.value = this.totals[`team_${team.id}`].value * (criterion.percentage / this.totalCriteriaPercentage);
                    ratings.push(rating);
                }

                // auto-save total score
                $.ajax({
                    url: `${this.$store.getters.appURL}/${this.$store.getters['auth/getUser'].userType}.php`,
                    type: 'POST',
                    xhrFields: {
                        withCredentials: true
                    },
                    data: {
                        ratings
                    },
                    success: (data, textStatus, jqXHR) => {
                        if (this.totals[`team_${team.id}`].loading) {
                            setTimeout(() => {
                                this.totals[`team_${team.id}`].loading = false;
                            }, 1000);
                        }
                    },
                    error: (error) => {
                        alert(`ERROR ${error.status}: ${error.statusText}`);
                    }
                })
            },
            openSubmitDialog() {
                // open dialog according to ratings
                for (let i = 0; i < this.teams.length; i++) {
                    if (!this.teams[i].disabled && (this.totals[`team_${this.teams[i].id}`].value < this.minRating || this.totals[`team_${this.teams[i].id}`].value > this.maxRating)) {
                        this.inspectDialog = true
                        this.submitDialog = false;
                        break;
                    } else {
                        this.submitDialog = true
                    }
                }
            },
            submitRatings() {
                // set loading state
                this.submitLoading = true;

                // prepare ratings array
                let ratings = [];
                for (let i = 0; i < this.teams.length; i++) {
                    const team = this.teams[i];
                    for (let criterion of this.criteria) {
                        const rating = this.ratings[`${this.event.slug}_${team.id}`][`${this.$store.getters['auth/getUser'].id}_${criterion.id}_${team.id}`];
                        ratings.push(rating);
                    }
                }

                // send data
                $.ajax({
                    url: `${this.$store.getters.appURL}/${this.$store.getters['auth/getUser'].userType}.php`,
                    type: 'POST',
                    xhrFields: {
                        withCredentials: true
                    },
                    data: {
                        ratings,
                        locking: true
                    },
                    success: (data, textStatus, jqXHR) => {
                        if (this.submitLoading) {
                            setTimeout(() => {
                                this.submitLoading = false
                                this.submitDialog = false;

                                // lock all ratings after submission
                                for (let i = 0; i < ratings.length; i++) {
                                    ratings[i].is_locked = true;
                                }

                                // lock all total scores for each score
                                for (let i = 0; i < this.teams.length; i++) {
                                    this.totals[`team_${this.teams[i].id}`].is_locked = true;
                                }
                            }, 600);
                        }
                    },
                    error: (error) => {
                        this.submitLoading = false
                        alert(`ERROR ${error.status}: ${error.statusText}`);
                    }
                })
            },
            move(x, y, callback, focus = true) {
                // move to input
                const nextInput = document.querySelector(`#${this.componentId} #input_${y}_${x}`);
                if (nextInput) {
                    if(nextInput.disabled) {
                        if(callback)
                            callback(x, y);
                    }
                    else {
                        if (focus)
                            nextInput.focus();
                        if (Number(nextInput.value) <= 0)
                            nextInput.select();
                    }
                }
            },
            moveDown(x, y) {
                // move to input below
                y += 1;
                if (y < this.teams.length)
                    this.move(x, y, this.moveDown);
            },
            moveUp(x, y) {
                // move to input above
                y -= 1;
                if (y >= 0)
                    this.move(x, y, this.moveUp);
            },
            moveRight(x, y) {
                // move to input to the right
                x += 1;
                if (x <= this.criteria.length)
                    this.move(x, y, this.moveRight);
            },
            moveLeft(x, y) {
                // move to input to the left
                x -= 1;
                if (x >= 0)
                    this.move(x, y, this.moveLeft);
            },
            updateCoordinates(x, y) {
                // get input coordinates
                this.coordinates.x = x;
                this.coordinates.y = y;
                this.move(x, y, null, false);
            }
        },

        mounted() {
            this.fetchScoreSheet();
        }
    }
</script>


<style scoped>
    tbody td, th {
        height: 64px !important;
    }

    tbody td {
        border-bottom: 1px solid #ddd;
        padding-top: 9px !important;
        padding-bottom: 9px !important;
    }

    #warning {
        animation: tilt-shaking 1ms linear infinite;
    }

    #remind {
        animation: tilt-shaking 1s linear infinite;
    }

    @keyframes tilt-shaking {
        0% {
            transform: rotate(0deg);
        }
        25% {
            transform: rotate(6deg);
        }
        50% {
            transform: rotate(0deg);
        }
        75% {
            transform: rotate(-6deg);
        }
        100% {
            transform: rotate(0deg);
        }
    }

    @keyframes shine {
        0% {
            background-position: 0% 50%;
        }
        50% {
            background-position: 100% 50%;
        }
        100% {
            background-position: 0% 50%;
        }
    }
</style>