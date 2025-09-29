<template>
  <div class="movie-card-new" @click="goToMovieDetail">
    <div class="movie-poster">
      <img
        :src="getImageUrl(movie.poster_url)"
        :alt="movie.name"
        class="poster-image"
        loading="lazy"
      />
      
      <!-- Remove button (dấu X) -->
      <button
        v-if="showRemoveButton"
        class="remove-button"
        @click.stop="removeMovie"
        :title="'Xóa ' + movie.name + ' khỏi danh sách đã lưu'"
      >
        <i class="bi bi-x"></i>
      </button>
      
      <!-- Gradient overlay luôn hiển thị -->
      <div class="movie-overlay-permanent">
        <!-- Episode badge ở góc trên phải -->
        <div class="episode-badge">
          {{ movie.episode_current }}
        </div>
        
        <!-- Movie info ở dưới -->
        <div class="movie-info-bottom">
          <h3 class="movie-title" :title="movie.name">{{ movie.name }}</h3>
          
          <div class="movie-rating">
            <div class="stars">
              <i
                v-for="star in 5"
                :key="star"
                :class="getStarClass(star, getRating())"
              ></i>
            </div>
            <span
              class="rating-score"
              :class="{ 'no-rating': getRating() === 0 }"
            >{{ formatRating(getRating()) }}</span>
          </div>
        </div>
      </div>
      
      <!-- Play button overlay chỉ hiện khi hover -->
      <div class="play-overlay">
        <div class="play-button">
          <i class="bi bi-play-circle"></i>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "MovieCardNew",
  props: {
    movie: {
      type: Object,
      required: true,
    },
    showRemoveButton: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
    getImageUrl(posterUrl) {
      if (!posterUrl) {
        return '';
      }
      let originalUrl;
      if (posterUrl.startsWith("http") || posterUrl.startsWith("//")) {
        originalUrl = posterUrl;
      } else {
        originalUrl = `https://phimimg.com/${posterUrl}`;
      }
      return `https://phimapi.com/image.php?url=${encodeURIComponent(originalUrl)}`;
    },

    getRating() {
       // Ưu tiên sử dụng rating từ movie.rating (đã được thêm vào movieInfo)
      if (this.movie.rating !== undefined && this.movie.rating !== null) {
        console.log('Using movie.rating:', this.movie.rating, 'for movie:', this.movie.name);
        return this.movie.rating;
      }

      // Fallback to tmdb.vote_average (for backward compatibility)
      if (this.movie.tmdb && this.movie.tmdb.vote_average) {
        console.log('Using tmdb.vote_average:', this.movie.tmdb.vote_average, 'for movie:', this.movie.name);
        return this.movie.tmdb.vote_average;
      }

      console.log('No rating found for movie:', this.movie.name, 'movie object:', this.movie);
      return 0;
    },

    formatRating(rating) {
      if (rating === 0) {
        return "0.0";
      }
      const rounded = Math.round(rating * 10) / 10;
      return rounded.toFixed(1);
    },

    getStarClass(starPosition, rating) {
      if (rating === 0) {
        return "bi bi-star no-rating";
      }

      const starRating = rating / 2;

      if (starPosition <= Math.floor(starRating)) {
        return "bi bi-star-fill";
      } else if (
        starPosition === Math.floor(starRating) + 1 &&
        starRating % 1 >= 0.5
      ) {
        return "bi bi-star-half";
      } else {
        return "bi bi-star";
      }
    },

    goToMovieDetail() {
      this.$router.push(`/phim/${this.movie.slug}`);
    },

    removeMovie() {
      console.log('Remove movie clicked for:', this.movie.slug);
      console.log('showRemoveButton prop:', this.showRemoveButton);
      this.$emit('remove-movie', this.movie.slug);
    },
  },
};
</script>

<style scoped>
.movie-card-new {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 15px;
  overflow: hidden;
  transition: all 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  cursor: pointer;
  position: relative;
}

.movie-card-new:hover {
  transform: translateY(-8px);
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.3);
  border-color: rgba(255, 107, 107, 0.3);
}

.movie-poster {
  position: relative;
  aspect-ratio: 2/3;
  overflow: hidden;
}

.poster-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.movie-card-new:hover .poster-image {
  transform: scale(1.05);
}

/* Overlay luôn hiển thị cho text và episode */
.movie-overlay-permanent {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.1) 0%,
    rgba(0, 0, 0, 0.2) 40%,
    rgba(0, 0, 0, 0.8) 100%
  );
  pointer-events: none; /* Cho phép click through */
}

/* Play overlay chỉ hiện khi hover */
.play-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.3);
  opacity: 0;
  transition: opacity 0.3s ease;
}

.movie-card-new:hover .play-overlay {
  opacity: 1;
}

/* Play button ở giữa */
.play-button {
  width: 60px;
  height: 60px;
  background: linear-gradient(45deg, #ff6b6b, #ffd93d);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 1.8rem;
  transform: scale(0.8);
  transition: transform 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.movie-card-new:hover .play-button {
  transform: scale(1);
}

/* Episode badge ở góc trên phải */
.episode-badge {
  position: absolute;
  top: 0.75rem;
  right: 0.75rem;
  background: linear-gradient(45deg, #ff6b6b, #ffd93d);
  color: white;
  padding: 0.3rem 0.6rem;
  border-radius: 12px;
  font-size: 0.70rem;
  font-weight: bold;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  pointer-events: none; /* Cho phép click through */
}

/* Movie info ở dưới */
.movie-info-bottom {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 1rem;
  background: linear-gradient(
    to top,
    rgba(0, 0, 0, 0.9) 0%,
    rgba(0, 0, 0, 0.7) 50%,
    transparent 100%
  );
  pointer-events: none; /* Cho phép click through */
}

.movie-title {
  font-size: 1.1rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
  color: #ffffff;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.8);
  line-height: 1.2;
}

.movie-rating {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.stars {
  display: flex;
  gap: 2px;
}

.stars i {
  color: #ffd93d;
  font-size: 0.85rem;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.5);
}

.stars .bi-star {
  color: rgba(255, 217, 61, 0.4);
}

.stars .bi-star.no-rating {
  color: rgba(255, 255, 255, 0.3);
}

.rating-score {
  font-size: 0.8rem;
  color: #ffd93d;
  font-weight: bold;
  background: rgba(0, 0, 0, 0.6);
  padding: 0.2rem 0.5rem;
  border-radius: 8px;
  border: 1px solid rgba(255, 217, 61, 0.3);
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.8);
}

.rating-score.no-rating {
  color: rgba(255, 255, 255, 0.7);
  background: rgba(0, 0, 0, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

/* Responsive Design */
@media (max-width: 768px) {
  .movie-info-bottom {
    padding: 0.8rem;
  }

  .movie-title {
    font-size: 1rem;
  }

  .stars i {
    font-size: 0.8rem;
  }

  .rating-score {
    font-size: 0.75rem;
    padding: 0.15rem 0.4rem;
  }

  .play-button {
    width: 40px;
    height: 40px;
    font-size: 1.2rem;
  }

  .remove-button {
    width: 28px;
    height: 28px;
    font-size: 0.9rem;
    background: linear-gradient(45deg, #ff6b6b, #ffd93d); /* Cùng màu với desktop */
  }
}

@media (max-width: 576px) {
  .movie-info-bottom {
    padding: 0.6rem;
  }

  .movie-title {
    font-size: 0.9rem;
    margin-bottom: 0.4rem;
  }

  .movie-rating {
    gap: 0.4rem;
  }

  .stars i {
    font-size: 0.75rem;
  }

  .rating-score {
    font-size: 0.7rem;
    padding: 0.1rem 0.3rem;
  }

  .play-button {
    width: 45px;
    height: 45px;
    font-size: 1.3rem;
  }

  .episode-badge {
    font-size: 0.65rem;
    padding: 0.2rem 0.4rem;
  }
}

@media (max-width: 400px) {
  .movie-info-bottom {
    padding: 0.5rem;
  }

  .movie-title {
    font-size: 0.85rem;
    margin-bottom: 0.3rem;
  }

  .movie-rating {
    gap: 0.3rem;
  }

  .stars i {
    font-size: 0.7rem;
  }

  .rating-score {
    font-size: 0.65rem;
    padding: 0.1rem 0.25rem;
  }

  .play-button {
    width: 40px;
    height: 40px;
    font-size: 1.2rem;
  }
}

/* Remove button (dấu X) */
.remove-button {
  position: absolute;
  top: 0.5rem;
  left: 0.5rem;
  width: 32px;
  height: 32px;
  background: linear-gradient(45deg, #ff6b6b, #ffd93d); /* Đổi màu giống episode badge */
  border: 2px solid rgba(255, 255, 255, 0.9); /* Tăng opacity viền trắng */
  border-radius: 50%;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 1000;
  opacity: 1;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  pointer-events: auto;
}

.remove-button:hover {
  background: linear-gradient(45deg, #ff8a8a, #ffdd5d); /* Màu hover sáng hơn */
  opacity: 1;
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(255, 107, 107, 0.4);
}

.remove-button i {
  font-size: 1rem;
  font-weight: bold;
}

.remove-button:active {
  transform: scale(0.95);
}
</style>
