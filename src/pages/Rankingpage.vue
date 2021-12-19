<template>
    <div>
        <button @click="goToPage('home')">
            Go to home
        </button>
        <h1>
            Ranking
        </h1>
        <table>
            <tr>
                <th>Ranking</th>
                <th>Song Title</th>
                <th>Total Points</th>
                <th>Spotify Link</th>
            </tr>
            <tr v-for="(song, i) in songs" :key="i">
                <td>{{i + 1}}</td>
                <td>{{song.title}}</td>
                <td>{{song.totalPoints}}</td>
                <td><iframe :src="song.spotify" height="80" width="100%" frameBorder="0"></iframe></td>
            </tr>
        </table>
    </div>
</template>

<script>
    export default {
        name: "Rankingpage",
        data(){
            return {
                songs: []
            }
        },
                mounted(){
            this.fetchSongs();
        },
        methods: {
            goToPage(page) {
                this.$emit("change-page", page);
            },
            fetchSongs() {
                // Get all songs
                const url = "http://webservies.be/eurosong/Songs";

                fetch(url)
                    .then((response) => {
                        return response.json();
                    })
                    .then((songs) => {
                        //call fetchVotes method to add totalPoints to songs
                        this.fetchVotes(songs);
                    });
            },

            fetchVotes(songs) {
                // Get all votes
                const url = "http://webservies.be/eurosong/Votes";

                fetch(url)
                    .then((response) => {
                        // response is text, so convert to json
                        return response.json();
                    })
                    .then((votes) => {
                        // loop over array songs with map method and then add totalPoints to songs with the sum of all vote points
                        songs.map(song => {
                            // create totalPoints variable to get the sum of all vote points
                            song.totalPoints = 0;
                            //filter all the votes out where the vote songID is the same as the song id, because there are multiple votes for 1 song, then add up all points for 1 song and add it to that song's object in the JSON (totalPoints)
                            votes.filter((vote) => vote.songID == song.id).forEach(element => {
                                if (element.songID == song.id) song.totalPoints += element.points
                            });
                            return song;
                        });
                        // sort songs from best to worst and change data of songs, so everything will get rerenderd;
                        this.songs = songs.sort((a, b) => b.totalPoints - a.totalPoints );
                    });
            }
        }
    }
</script>