<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <div class="q-pa-md">
        <div class="row justify-between q-mb-md">
          <q-btn color="green" label="Post" @click="showPostModel = true" />
          <q-input
            outlined
            dense
            placeholder="Search Bar"
            class="col-8"
            v-model="searchTerm"
          />
          <q-btn-dropdown color="green" label="Sort">
            <q-list>
              <q-item clickable v-close-popup @click="sortPosts('newest')">
                <q-item-section>Newest</q-item-section>
              </q-item>
              <q-item clickable v-close-popup @click="sortPosts('oldest')">
                <q-item-section>Oldest</q-item-section>
              </q-item>
            </q-list>
          </q-btn-dropdown>
        </div>


        <!-- Post Model -->
        <q-dialog v-model="showPostModel">
          <q-card style="width: 500px; max-height: 650px;">
            <q-card-section class="bg-green text-white">
              <div class="text-h5 row items-center justify-between">Stray Animal Post
                <q-btn icon="close" flat round dense v-close-popup />
              </div>
            </q-card-section>

            <q-card-section>
              <div class="text-center q-mb-md">
                <q-img v-if="postImage" :src="postImage" style="max-width: 200px; max-height: 200px;" />
              </div>
              <div class="text-center q-mb-md">
                <q-btn color="green" label="Upload Image" @click="openFileInput" class="q-mt-md" />
              </div>
              <input type="file" style="display: none" ref="fileInput" accept="image/*" @change="handleFileUpload" />
            </q-card-section>

            <q-card-section>
              <q-input v-model="animalName" label="Animal's Name" outlined dense />
            </q-card-section>
            <q-card-section>
              <q-input v-model="description" label="Description" outlined dense type="textarea" />
            </q-card-section>

            <q-card-section>
              <q-input v-model="locationName" label="Location Name" outlined dense />
            </q-card-section>
            <q-card-section>
              <div class="text-center q-mb-md">
                <q-btn color="green" label="Mark The Location" @click="openMapModel" class="q-mt-md" />
              </div>
            </q-card-section>

            <q-card-actions align="right">
              <q-btn color="green" label="Submit" @click="submitPost" />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <!-- Location Model -->
        <q-dialog v-model="showMapModel" @after-show="initializeMap">
          <q-card style="width: 500px; height: 500px;">
            <q-card-section class="bg-green text-white">
              <div class="text-h5 row items-center justify-between">
                Mark The Location of Stray Animals
                <q-btn icon="close" flat round dense @click="showMapModel = false" />
              </div>
            </q-card-section>
            <q-card-section>
              <div id="map" style="height: 400px;"></div>
            </q-card-section>
          </q-card>
        </q-dialog>

        <!-- Post Display -->
        <div class="row q-col-gutter-md">
          <div
            class="col-12 col-sm-6 col-md-4"
            v-for="post in filteredPosts"
            :key="post.id"
          >
            <q-card @click="showDetails(post)" class="my-card">
              <q-img :src="post.image" class="fixed-height-img"/>
              <q-card-section class="bg-green text-white">
                <div class="row no-wrap items-center">
                  <q-avatar>
                    <img :src="post.user_img" />
                  </q-avatar>
                  <div class="q-ml-sm text-weight-bold">{{ post.author }}</div>
                </div>
              </q-card-section>
              <q-card-section>
                <div class="text-h6">{{ post.title }}</div>
                <div class="text-body2" v-if="post.locationName" style="font-size: 0.85rem; color: gray;">
                  Found at: {{ post.locationName }}
                </div>
                <div class="text-body1 multiline-truncate">{{ post.description }}</div>
              </q-card-section>
            </q-card>
          </div>
        </div>

        <!-- Post details -->
        <q-dialog v-model="detailsDialog" no-refocus no-focus>
          <q-card class="my-card">
            <q-card-section>
              <q-img v-if="currentPost.image" :src="currentPost.image" />
            </q-card-section>

            <q-card-section class="bg-green text-white">
              <div class="row items-center justify-between">
                <div class="flex no-wrap items-center">
                  <q-avatar>
                    <img :src="currentPost.user_img" />
                  </q-avatar>
                  <div class="q-ml-sm text-h6">{{ currentPost.author }}</div>
                </div>
                <div v-if="currentPost.user_id == userId">
                  <q-btn icon="edit" flat round dense @click="openEditDialog(currentPost)" />
                  <q-btn icon="delete" flat round dense @click="confirmDelete(currentPost)" />
                </div>
              </div>
            </q-card-section>

            <q-card-section class="row items-center justify-between custom-section">
              <div class="text-h6 col">{{ currentPost.title }}</div>
              <q-btn v-if="currentPost.user_id == userId" class="mark-adopted" flat label="Mark Adopted" color="green" @click="markAsAdoptedDialog(currentPost)" />
            </q-card-section>

            <q-card-section v-if="currentPost.locationName" class="custom-description">
              <div class="text-body2" style="font-size: 0.85rem; color: gray;">
                {{ currentPost.locationName }}
              </div>
            </q-card-section>

            <q-card-section class="custom-description">
              <div>{{ currentPost.description }}</div>
            </q-card-section>

            <q-card-actions v-if="currentPost.user_id == userId">
              <q-btn color="green" label="Chat List" @click="openChatDialog"/>
              <q-btn color="green" label="Location" @click="openLocationDialog(currentPost)"/>
            </q-card-actions>

            <q-card-actions v-else>
              <q-btn color="green" label="Chat" @click="openChatDialog"/>
              <q-btn color="green" label="Location" @click="openLocationDialog(currentPost)"/>
            </q-card-actions>
          </q-card>
        </q-dialog>

        <!-- Location Map Dialog -->
        <q-dialog v-model="showLocationDialog" @after-show="initializePostMap">
          <q-card style="width: 500px; height: 500px;">
            <q-card-section class="bg-green text-white">
              <div class="text-h5 row items-center justify-between">
                Location of Stray Animal
                <q-btn icon="close" flat round dense @click="showLocationDialog = false" />
              </div>
            </q-card-section>
            <q-card-section>
              <div id="post-map" style="height: 400px;"></div>
            </q-card-section>
          </q-card>
        </q-dialog>

        <!-- Chat Dialog -->
        <q-dialog v-model="chatDialog" @hide="resetChatDialog">
          <q-card style="width: 600px; max-height: 650px;">
            <q-card-section class="bg-green text-white">
              <div class="text-h5 row items-center justify-between">
                <q-avatar>
                    <img :src="currentPost.user_img" />
                </q-avatar>
                Chat with {{ currentPost.author }}
                <q-btn icon="close" flat round dense v-close-popup @click="closeChatDialog" />
              </div>
            </q-card-section>

            <!-- Chat List Section for Post Owner -->
            <q-card-section v-if="currentPost.user_id == userId">
              <q-list v-if="chatList.length > 0">
                <q-item v-for="chat in chatList" :key="chat.chat_id" clickable @click="openChatMessages(chat.sender_id)">
                  <q-item-section>{{ chat.sender_name }}</q-item-section>
                  <q-item-section>{{ chat.message }}</q-item-section>
                </q-item>
              </q-list>
              <div v-else class="text-h5 row items-center justify-center">
                <p>No one has sent a message yet.</p>
              </div>
            </q-card-section>

            <!-- Chat Box Section -->
            <q-card-section v-else class="chat-box">
              <div v-if="chatMessages.length > 0">
                <div v-for="message in chatMessages" :key="message.id" 
                  :class="{'message-sent': message.sender_id == userId, 'message-received': message.receiver_id == userId}">
                  <div class="message-bubble">
                    <div class="message-sender">{{ message.sender_name }}:</div>
                    <div class="message-text">{{ message.message }}</div>
                    <div class="message-timestamp">{{ new Date(message.timestamp).toLocaleString() }}</div>
                  </div>
                </div>
              </div>
              <div v-else>
                <p>No messages yet.</p>
              </div>

              <q-card-section class="chat-input">
                <q-input v-model="newMessage" placeholder="Type a message" outlined dense @keyup.enter="sendMessage" class="chat-input-box"/>
                <q-btn label="Send" color="green" @click="sendMessage" />
              </q-card-section>
            </q-card-section>
          </q-card>
        </q-dialog>

        <!-- Edit Post Dialog -->
        <q-dialog v-model="showEditDialog">
          <q-card style="width: 500px">
            <q-card-section class="bg-green text-white">
              <div class="text-h6 row items-center justify-between">Edit Post
                <q-btn icon="close" flat round dense v-close-popup />
              </div>
            </q-card-section>
            <q-card-section>
              <!-- Image preview and upload input -->
              <div class="text-center q-mb-md">
                <q-img v-if="editPostImage" :src="editPostImage" style="max-width: 300px; max-height: 300px;" />
              </div>
              <div class="text-center q-mb-md">
                <q-btn color="green" label="Upload Image" @click="openEditFileInput" class="q-mt-md" />
                <!-- Remove hidden attribute and use styles to hide the input -->
                <input type="file" ref="editFileInput" accept="image/*" @change="handleEditFileUpload" style="display: none;" />
              </div>
              <q-input v-model="postToEdit.title" label="Animal's Name" />
              <q-input v-model="postToEdit.description" label="Description" type="textarea" />
              <q-input v-model="postToEdit.locationName" label="Location Name" />
              <div class="text-center q-mb-md">
                <q-btn color="green" label="Mark The Location" @click="openEditMapModel" class="q-mt-md" />
              </div>
            </q-card-section>
            <q-card-actions align="right">
              <q-btn label="Cancel" color="black" flat @click="showEditDialog = false" />
              <q-btn label="Save" color="green" @click="editPost" />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <!-- Edit Map Model -->
        <q-dialog v-model="showEditMapModel" @after-show="initializeEditMap">
          <q-card style="width: 500px; height: 500px;">
            <q-card-section class="bg-green text-white">
              <div class="text-h5 row items-center justify-between">
                Edit The Location of Stray Animals
                <q-btn icon="close" flat round dense @click="showEditMapModel = false" />
              </div>
            </q-card-section>
            <q-card-section>
              <div id="edit-map" style="height: 400px;"></div>
            </q-card-section>
          </q-card>
        </q-dialog>

        <!-- Delete Confirmation Dialog -->
        <q-dialog v-model="showDeleteConfirmDialog">
          <q-card>
            <q-card-section class="row items-center">
              <q-avatar icon="warning" color="amber" text-color="white" />
              <span class="q-ml-sm">Are you sure you want to delete this post?</span>
            </q-card-section>
            <q-card-actions align="right">
              <q-btn label="Cancel" color="black" flat @click="showDeleteConfirmDialog = false" />
              <q-btn label="Yes, Delete" color="negative" @click="deletePost" />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <!-- Mark Adopted -->
        <q-dialog v-model="showConfirmationDialog">
          <q-card>
            <q-card-section>
              Have the stray animals in this post been adopted?
            </q-card-section>
            <q-card-actions align="right">
              <q-btn flat label="Cancel" color="negative" @click="showConfirmationDialog = false" />
              <q-btn flat label="Confirm" color="green" @click="confirmAdoption" />
            </q-card-actions>
          </q-card>
        </q-dialog>


      </div>
    </q-page-container>
  </q-layout>
</template>

<script>
import { useLoginUserStore } from "../stores/loginUserStore";
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import { ref, uploadBytes, getDownloadURL } from "firebase/storage"; // Firebase imports
import { storage } from "../firebaseConfig";

const BASE_IMAGE_URL = 'https://firebasestorage.googleapis.com/v0/b/final-project-142d2.appspot.com/o/';

export default {
  data() {
    return {
      currentPost: null,
      detailsDialog: false,
      showPostModel: false,
      postImage: null,
      animalName: '',
      description: '',
      locationName: '',
      latitude: null,
      longitude: null,
      map: null,
      marker: null,
      showMapModel: false,
      showEditMapModel: false,
      showLocationDialog: false,
      postMap: null,
      postMarker: null,
      searchTerm: '',
      posts: [],
      chatDialog: false,
      chatList: [],
      chatMessages: [],
      newMessage: '',
      showEditDialog: false,
      showDeleteConfirmDialog: false,
      postImageFile: null,
      editPostImageFile: null,
      editPostImage: null,
      postToEdit: {},
      postToDelete: null,
      showConfirmationDialog: false,
      postToBeMarkedAdopted: null,
      loginUserStore: useLoginUserStore(),
    }
  },
  computed: {
    filteredPosts() {
      return this.posts.filter((post) => {
        const searchTermLower = this.searchTerm.toLowerCase()
        return post.adopted === 0 && // Add this line to check for non-adopted animals
        (post.title.toLowerCase().includes(searchTermLower) ||
        post.description.toLowerCase().includes(searchTermLower) ||
        post.author.toLowerCase().includes(searchTermLower));
      })
    },
    userId() {
      return this.loginUserStore.userid;
    },
  },
  methods: {
    openFileInput() {
      this.$refs.fileInput.click()
    },
    openEditFileInput() {
      this.$refs.editFileInput.click();
    },
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        this.postImageFile = file;
        this.postImage = URL.createObjectURL(file);
      }
    },
    handleEditFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        this.editPostImageFile = file;
        this.editPostImage = URL.createObjectURL(file);
      }
    },
    openMapModel() {
      this.showMapModel = true;
      this.$nextTick(() => {
        setTimeout(() => {
          this.initializeMap();
        }, 300);
      });
    },
    openEditMapModel() {
      this.showEditMapModel = true;
      this.$nextTick(() => {
        this.initializeEditMap();
      });
    },
    initializeMap() {
      const mapContainer = document.getElementById("map");

      if (!mapContainer) {
        console.error("Map container not found. Retrying...");
        return;
      }

      // Remove the map instance if it already exists to avoid multiple instances
      if (this.map) {
        this.map.remove();
        this.map = null;
      }

      // Initialize the map
      const latitude = this.latitude || 7.8804;
      const longitude = this.longitude || 98.3923;
      
      this.map = L.map(mapContainer).setView([latitude, longitude], 13);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.map);

      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            const { latitude, longitude } = position.coords;
            this.latitude = latitude;
            this.longitude = longitude;

            // Center the map on the user's current location and add a marker
            this.map.setView([latitude, longitude], 13);

            if (this.marker) {
              this.marker.addTo(this.map).setLatLng([latitude, longitude]);
            } else{
              this.marker = L.marker([latitude, longitude]).addTo(this.map);
            }
          },
          (error) => {
            console.error("Geolocation error:", error);
            // Fall back to default location
            this.map.setView([7.8804, 98.3923], 13);
          }
        );
      }
      
      // Add click event to allow users to place a new marker or move the existing marker
      this.map.on('click', (e) => {
        const { lat, lng } = e.latlng;
        this.latitude = lat;
        this.longitude = lng;

        // If the marker exists, move it, otherwise create a new one
        if (this.marker) {
          this.marker.setLatLng([lat, lng]);
        } else {
          this.marker = L.marker([lat, lng]).addTo(this.map);
        }
      });
    },
    async initializeEditMap() {
      const mapElement = document.getElementById("edit-map");
      if (!mapElement) {
        console.error("Edit map container not found");
        return;
      }

      try {
        // Make an API call to get the post's location by its ID
        const response = await this.$api.get(`/map-location/${this.postToEdit.id}`);
        const postLocation = response.data; // Assuming the API returns { latitude, longitude }

        // Use the fetched latitude and longitude, or fall back to default values if not available
        const latitude = postLocation.latitude || 7.8804;
        const longitude = postLocation.longitude || 98.3923;

        // Remove the previous map if it exists
        if (this.postToEdit.map) {
          this.postToEdit.map.remove();
          this.postToEdit.map = null;
        }

        // Initialize a new map centered on the post's location
        this.postToEdit.map = L.map(mapElement).setView([latitude, longitude], 13);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.postToEdit.map);

        // Place the marker at the post's location or default location
        if (!this.postToEdit.marker) {
          this.postToEdit.marker = L.marker([latitude, longitude]).addTo(this.postToEdit.map);
        } else {
          this.postToEdit.marker.addTo(this.postToEdit.map).setLatLng([latitude, longitude]);
        }

        // Allow the user to update the marker by clicking on the map
        this.postToEdit.map.on('click', (e) => {
          const { lat, lng } = e.latlng;
          this.postToEdit.latitude = lat;
          this.postToEdit.longitude = lng;

          if (this.postToEdit.marker) {
            this.postToEdit.marker.setLatLng([lat, lng]);
          } else {
            this.postToEdit.marker = L.marker([lat, lng]).addTo(this.postToEdit.map);
          }
        });
        
      } catch (error) {
        console.error('Error fetching post location:', error);
        // Fall back to default coordinates in case of error
        const defaultLat = 7.8804;
        const defaultLng = 98.3923;

        // Reinitialize map with default location
        this.postToEdit.map = L.map(mapElement).setView([defaultLat, defaultLng], 13);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.postToEdit.map);
      }
    },

    openLocationDialog(post) {
      this.currentPost = post;
      this.showLocationDialog = true;
      this.$nextTick(() => {
        this.initializePostMap();
      });
    },
    async initializePostMap() {
      const mapElement = document.getElementById("post-map");

      if (!mapElement) {
        console.error("Map container not found");
        return;
      }

      // Remove any existing map instance
      if (this.postMap) {
        this.postMap.remove();
      }

      try {
        // Fetch the post's location using the API call, similar to initializeEditMap
        const response = await this.$api.get(`/map-location/${this.currentPost.id}`);
        const postLocation = response.data; // Assuming the API returns { latitude, longitude }

        // Ensure latitude and longitude values are available, fallback to default values if not
        const latitude = postLocation.latitude || 7.8804; // Default latitude (Phuket)
        const longitude = postLocation.longitude || 98.3923; // Default longitude

        // Initialize the map
        this.postMap = L.map(mapElement).setView([latitude, longitude], 13);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.postMap);

        // Add a marker to the map at the fetched coordinates
        if (latitude && longitude) {
          this.postMarker = L.marker([latitude, longitude]).addTo(this.postMap);
        }
      } catch (error) {
        console.error('Error fetching post location:', error);

        // Fallback to default coordinates if the API call fails
        const defaultLat = 7.8804;
        const defaultLng = 98.3923;

        this.postMap = L.map(mapElement).setView([defaultLat, defaultLng], 13);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.postMap);
      }
    },
    sortPosts(order) {
      if (order === 'newest') {
        this.posts.sort((a, b) => new Date(b.created_at) - new Date(a.created_at));
      } else if (order === 'oldest') {
        this.posts.sort((a, b) => new Date(a.created_at) - new Date(b.created_at));
      }
    },
    showDetails(post) {
      console.log('Post user ID:', post.user_id, 'Logged-in user ID:', this.userId);
      this.currentPost = post;
      this.detailsDialog = true;
    },
    async submitPost() {
      try {
        if (!this.postImageFile) {
          throw new Error('Please upload an image.');
        }

        // Upload image to Firebase Storage
        const fileRef = ref(storage, `posts/${Date.now()}-${this.postImageFile.name}`);
        await uploadBytes(fileRef, this.postImageFile);
        const imageUrl = await getDownloadURL(fileRef);

        // // Post the image file to your backend
        // const fileResponse = await this.$api.post('/file/upload', formData, {
        //   headers: {
        //     'Content-Type': 'multipart/form-data'
        //   },
        // });

        // // Verify that the file upload response is as expected
        // if (!fileResponse.data.uploadFileName) {
        //   throw new Error('File upload did not return a filename.');
        // }

         // Submit the post to the backend
        const postResponse = await this.$api.post('/post/', {
          title: this.animalName,
          description: this.description,
          image: imageUrl, // Firebase image URL
          user_id: this.userId, // Need to have the user's id
        });

        const newPost = postResponse.data;

        await this.$api.post('/map-location/', {
          post_id: newPost.id,
          latitude: this.latitude,
          longitude: this.longitude,
          description: this.locationName
        });

        this.$q.notify({
          type: 'positive',
          message: 'Post successfully.'
        });

        // Refresh the list after submitting
        await this.getData();

        // Reset the form and close the model
        this.closePostModel();
      } catch (error) {
        console.error('Error:', error.response ? error.response.data : error.message);
        this.$q.notify({
          type: 'negative',
          message: error.message || 'Failed to create post.'
        });
      }
    },
    closePostModel() {
      this.showPostModel = false;
      this.postImageFile = null;
      this.postImage = '';
      this.animalName = '';
      this.description = '';
      this.locationName = '';
      // Reset the file input if needed
      if (this.$refs.fileInput) {
        this.$refs.fileInput.value = '';
      }
    },
    async getData() {
      try {
        const response = await this.$api.get('/post/all');
        console.log('Posts received from backend:', response.data);
        this.posts = response.data
          .map(async post => {
            const locationResponse = await this.$api.get(`/map-location/${post.id}`);
            const imageUrl = post.image;

            return {
              ...post,
              author: post.author || 'Unknown',
              description: post.description || 'No description',
              image: imageUrl,
              user_id: post.user_id,
              user_img: post.user_img && !post.user_img.startsWith('http') ? `${BASE_IMAGE_URL}${post.user_img}` : post.user_img,
              locationName: locationResponse.data.locationName || '',  // Include the location name from the location API response
            };
          });

        this.posts = await Promise.all(this.posts);  // Wait for all location fetches to resolve
        this.posts.sort((a, b) => new Date(b.created_at) - new Date(a.created_at)); // Sort by latest
      } catch (error) {
        this.$q.notify({
          type: 'negative',
          message: 'Error fetching posts: ' + error.message
        });
      }
    },
    openEditDialog(post) {
      this.postToEdit = { ...post }; // Make a copy to avoid mutating the original post
      this.editPostImage = post.image;
      this.showEditDialog = true;
    },

    confirmDelete(post) {
      this.postToDeleteId = post.id;
      this.showDeleteConfirmDialog = true;
    },

    async editPost() {
      if (!this.postToEdit.id) {
        this.$q.notify({
          type: 'negative',
          message: 'Post ID is not available.'
        });
        return;
      }
      try {
        let imageUrl = this.postToEdit.image;
        // Check if a new image file has been selected
        if (this.editPostImageFile) {
          // Upload the new image first
          const fileRef = ref(storage, `posts/${Date.now()}-${this.editPostImageFile.name}`);
          await uploadBytes(fileRef, this.editPostImageFile);
          imageUrl = await getDownloadURL(fileRef);
        }

        // Update the post details, using Vue.set for reactivity
        const dataToUpdate = {
          title: this.postToEdit.title,
          description: this.postToEdit.description,
          image: imageUrl,
        };

        await this.$api.put(`/post/${this.postToEdit.id}`, dataToUpdate);

        if (this.postToEdit.latitude !== this.latitude || this.postToEdit.longitude !== this.longitude){
            if (this.latitude && this.longitude) {
                const updatedLocation = {
                    latitude: this.latitude,
                    longitude: this.longitude,
                    description: this.locationName || this.postToEdit.locationName
                };
                await this.$api.put(`/map-location/${this.postToEdit.id}`, updatedLocation);
            }
        }

        // Post was updated successfully
        this.$q.notify({
          type: 'positive',
          message: 'Post updated successfully.'
        });
        this.detailsDialog = false;
        this.showEditDialog = false;
        await this.getData(); // Refresh the list
      } catch (error) {
        // Handle errors here
        console.error('Error:', error);
        this.$q.notify({
          type: 'negative',
          message: 'Failed to update post.'
        });
      }
    },

    updatePostWithNewImage() {
      // Proceed to call your API to update the post details
      const dataToUpdate = {
        title: this.postToEdit.title,
        description: this.postToEdit.description,
        image: this.postToEdit.image,
      };

      this.$api.put(`/post/${this.postToEdit.id}`, dataToUpdate)
        .then(() => {
          // Post was updated successfully
          this.$q.notify({
            type: 'positive',
            message: 'Post updated successfully.'
          });
          this.showEditDialog = false;
          this.getData(); // Refresh the list
        })
        .catch(error => {
          // Handle errors here
          console.error('Error updating post:', error);
          this.$q.notify({
            type: 'negative',
            message: 'Failed to update post.'
          });
        });
    },

    deletePost() {
      // Check if we have a valid post ID
      if (!this.postToDeleteId) {
        this.$q.notify({
          type: 'negative',
          message: 'Post ID is not available.'
        });
        return;
      }

      // Perform the API call to delete the post
      this.$api.delete(`/post/${this.postToDeleteId}`)
        .then(() => {
          this.$q.notify({
            type: 'positive',
            message: 'Post deleted successfully.'
          });
          this.detailsDialog = false;
          this.showDeleteConfirmDialog = false; // Hide the confirmation dialog
          this.postToDeleteId = null; // Reset the postToDeleteId
          this.getData();
        }).catch((error) => {
          console.error('Error:', error);
          this.$q.notify({
            type: 'negative',
            message: 'Failed to delete post.'
          });
        });
    },
    markAsAdoptedDialog(post) {
      this.postToBeMarkedAdopted = post;
      this.showConfirmationDialog = true;
    },
    async confirmAdoption() {
      try {
        // Make sure you have the post ID to update
        if (!this.postToBeMarkedAdopted) throw new Error('Post not specified.');

        // Here, you would make an API call to update the adopted status
        await this.$api.put(`/post/${this.postToBeMarkedAdopted.id}/mark`, {
          adopted: 1,
        });

        this.$q.notify({
          type: 'positive',
          message: 'The animal has been marked as adopted.',
        });

        // Close the confirmation dialog
        this.detailsDialog = false;
        this.showConfirmationDialog = false;

        // Refresh the posts list
        await this.getData();
      } catch (error) {
        console.error('Error:', error);
        this.$q.notify({
          type: 'negative',
          message: 'Failed to mark as adopted.',
        });
      }
    },
    openChatDialog() {
      this.chatDialog = true;
      if (this.currentPost.user_id == this.userId) {
        if (this.chatList.length === 0) {
          this.loadChatList();
        }
      } else {
        this.loadChatMessages(this.userId, this.currentPost.user_id);
      }
    },
    closeChatDialog() {
      this.chatDialog = false;
      this.chatMessages = [];
    },
    resetChatDialog() {
      this.chatMessages = [];
    },

    async loadChatList() {
      try {
        const response = await this.$api.get(`/chats/post/${this.currentPost.id}`);
        this.chatList = response.data;
      } catch (error) {
        console.error('Error loading chat list:', error);
      }
    },
    async loadChatMessages(senderId, receiverId) {
      try {
        const response = await this.$api.get(`/chats/messages/${this.currentPost.id}/${senderId}/${receiverId}`);
        this.chatMessages = response.data;
      } catch (error) {
        console.error('Error loading chat messages:', error);
      }
    },
    async sendMessage() {
      if (!this.newMessage.trim()) return;
      try {
        await this.$api.post('/chats/send', {
          post_id: this.currentPost.id,
          sender_id: this.userId,
          receiver_id: this.currentPost.user_id,
          message: this.newMessage
        });
        this.newMessage = '';
        this.loadChatMessages(this.userId, this.currentPost.user_id);
      } catch (error) {
        console.error('Error sending message:', error.response ? error.response.data : error.message);
      }
    },
  },
  mounted() {
    this.getData();
  },
}
</script>

<style scoped>
.my-card {
  width: 400px;
  max-width: 100%;
  max-height: 100%;
}

.fixed-height-img {
  height: 250px;
}

.multiline-truncate {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2; /* Show only 2 lines */
  overflow: hidden; /* Hide overflow */
  text-overflow: ellipsis; /* Show ellipsis */
  white-space: normal; /* Ensure text wraps */
}

#map {
  width: 100%;
  height: 100%;
}

.mark-adopted {
  padding: 4px 0px;
  margin-left: auto;
}

.custom-section {
  margin-bottom: 0; /* Remove the bottom margin from the title section */
}

.custom-description {
  padding-top: 0; /* Remove the top padding from the description section */
  margin-top: 0; /* Remove the top margin to bring it closer to the title */
  overflow-y: auto;
}

.chat-box {
  max-height: 400px;
  overflow-y: auto;
  padding: 10px;
}

.message-sent {
  align-self: flex-end;
  text-align: right;
  margin-left: auto;
}

.message-received {
  align-self: flex-start;
  text-align: left;
}

.message-bubble {
  display: inline-block;
  max-width: 80%;
  padding: 10px;
  border-radius: 20px;
  margin-bottom: 10px;
  font-size: 0.9rem;
}

.message-sent .message-bubble {
  background-color: #dcf8c6; /* Light green bubble for sent messages */
}

.message-received .message-bubble {
  background-color: #f1f0f0; /* Light grey bubble for received messages */
}

.message-sender {
  font-weight: bold;
  margin-bottom: 2px;
}

.chat-input {
  display: flex;
  gap: 5px;
  margin-top: 10px;
  align-items: center;
}

.chat-input-box {
  flex-grow: 1;
}

</style>
