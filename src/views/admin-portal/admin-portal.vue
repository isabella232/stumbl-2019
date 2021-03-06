<style lang="scss">
@import './admin-portal.scss';
</style>

<template>
  <div>
    <transition name="fade">
      <AlertBanner
        v-if="bannerActive"
        :color="bannerColor"
        class="portal__alert-banner"
      >{{ bannerMessage }}</AlertBanner>
    </transition>
    <Header></Header>
    <div class="portal">
      <h2 class="header-2">Startup crawl company admin</h2>
      <a class="portal__page-link" :href="'/venue/' + venueId">
        <span class="portal__page-link">View Your Company Page</span>
      </a>
      <p>Edit your company details below. This will be visible to crawl participants when they check in to your location.</p>
      <label class="portal__label">
        Company name
        <input
          type="text"
          class="portal__input"
          :class="{'invalid-field': !validName}"
          name="company-name"
          placeholder="Company Name"
          v-model="venue.name"
        />
      </label>
      <label class="portal__label">
        Address
        <input
          type="text"
          class="portal__input"
          :class="{'invalid-field': !validAddress}"
          name="address"
          placeholder="Company Address"
          v-model="venue.address"
        />
      </label>
      <label class="portal__label">
        Unit #
        <input
          type="text"
          class="portal__input"
          name="address_2"
          placeholder="Company Address Cont."
          v-model="venue.address_2"
        />
      </label>
      <label class="portal__label">
        City
        <input
          type="text"
          class="portal__input"
          name="city"
          placeholder="City"
          v-model="venue.city"
        />
      </label>
      <label class="portal__label">
        State
        <input
          type="text"
          class="portal__input"
          name="state"
          placeholder="State"
          v-model="venue.state"
        />
      </label>
      <label class="portal__label">
        Zip Code
        <input
          type="text"
          class="portal__input"
          name="zip"
          placeholder="Zip Code"
          v-model="venue.zip"
        />
      </label>
      <label class="portal__label">
        Website
        <input
          type="text"
          class="portal__input"
          name="website"
          placeholder="Company Website"
          v-model="venue.website"
        />
      </label>
      <label for="company-image" class="portal__label">Featured Image</label>
      <div class="portal__image-container">
        <img
          v-if="imageLoading"
          class="portal__image-loading-spinner"
          src="../../images/rolling-loader.gif"
        />
        <img
          v-else
          class="portal__image"
          name="company-image"
          id="company-image"
          ref="company_img"
          v-lazy="displayImage"
        />
      </div>
      <label class="portal__image-input-label">
        <input
          @change="imageUpdated"
          type="file"
          accept="image/*"
          name="image-input"
          class="portal__image-input"
        />
        Select New Image
      </label>
      <label for="about-company" class="portal__label">About (What does your company do?)</label>
      <textarea
        id="about-company"
        placeholder="Company Description"
        class="portal__textarea"
        name="about-company"
        v-model="venue.description"
      ></textarea>
      <label for="special-instructions" class="portal__label">What will your event feature?</label>
      <textarea
        id="features"
        placeholder="Features"
        class="portal__textarea"
        name="features"
        v-model="venue.features"
      ></textarea>
      <label for="special-instructions" class="portal__label">Special instructions for attendees</label>
      <textarea
        id="special-instructions"
        placeholder="Special Instructions"
        class="portal__textarea"
        name="special-instructions"
        v-model="venue.special_instructions"
      ></textarea>
      <div @click="updateVenue()">
        <Button class="portal__button" title="save changes"></Button>
      </div>
      <router-link to="/home">
        <p class="portal__route-exit">Cancel and discard changes</p>
      </router-link>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue } from 'vue-property-decorator';
import Component from 'vue-class-component';
import Button from '@/components/button/button-component.vue';
import AlertBanner from '@/components/alert-banner/alert-banner-component.vue';
import Header from '@/components/header/header-component.vue';
import { DEFAULT_VENUE } from '@/services/venue-service';
import { Venue } from '@/models/venue-model';
import ApiService from '@/services/api-service';
import { Watch } from 'vue-property-decorator';

@Component({
  components: {
    Button,
    Header,
    AlertBanner,
  },
  props: {
    allVenues: Array,
    venueId: String,
  },
})
export default class AdminPortal extends Vue {
  public apiService = new ApiService();
  public venue: Venue = DEFAULT_VENUE;
  public venueId!: string;
  public allVenues!: Venue[];
  public bannerActive: boolean = false;
  public bannerMessage: string = 'Changes Saved!';
  public bannerColor: string = 'green';
  public imageChanged: boolean = false;
  public newImage: any;
  public newImageToUpload: any;
  public imageLoading: boolean = false;

  @Watch('allVenues')
  public handleUpdate() {
    const copy = this.allVenues.find((venue) => venue.id === this.venueId);
    const freshCopy = Object.assign({}, copy);
    this.venue = freshCopy;
  }

  @Watch('bannerActive')
  public promptSuccessMessage(value: boolean) {
    if (value) {
      window.scrollTo(0, 0);
      setTimeout(() => {
        this.bannerActive = false;
        this.bannerColor = 'green';
        this.bannerMessage = 'Changes Saved!';
      }, 3500);
    }
  }

  public async beforeMount() {
    // Needed for hard refresh of page.
    const copy = this.allVenues.find((venue) => venue.id === this.venueId);
    const freshCopy = Object.assign({}, copy);
    this.venue = freshCopy;
  }

  get validAddress() {
    return !!this.venue.address;
  }

  get validName() {
    return !!this.venue.name;
  }

  get validForm() {
    return this.validAddress && this.validName;
  }

  get validImage() {
    // Checks to see if the new image is a blob
    return this.newImage.includes('blob');
  }

  get displayImage() {
    return this.imageChanged
      ? this.newImage
      : this.venue.company_img_url
      ? this.venue.company_img_url
      : require('../../images/company-images/dsw-default-company.png');
  }

  public async imageUpdated(e: Event) {
    this.imageLoading = true;
    const target = e.target as HTMLInputElement;
    const response = await this.apiService.updateCompanyImage(
      this.venue,
      (target.files as FileList)[0],
    );

    this.venue.company_img_url = response.data.attributes.company_img_url;
    this.imageLoading = false;
  }

  public async updateVenue() {
    if (!this.validForm) {
      this.bannerMessage = 'Please complete required fields.';
      this.bannerColor = 'red';
      this.bannerActive = true;
    } else {
      const response = await this.apiService.updateVenue(this.venue);
      if (response.data) {
        this.bannerActive = true;
        this.venue = {
          id: response.data.id,
          ...response.data.attributes,
        };
      } else {
        this.bannerActive = true;
        this.bannerMessage =
          'Error saving.  You might not have permission to edit this venue.';
        this.bannerColor = 'red';
      }
    }
  }

  public async submitImage() {
    if (!this.validImage) {
      this.bannerMessage = 'Please provide valid image.';
      this.bannerColor = 'red';
      this.bannerActive = true;
    } else if (!this.newImage) {
      this.bannerMessage = 'Please provide new image.';
      this.bannerColor = 'red';
      this.bannerActive = true;
    } else {
      const response = await this.apiService.updateCompanyImage(
        this.venue,
        this.newImageToUpload,
      );
    }
  }
}
</script>
