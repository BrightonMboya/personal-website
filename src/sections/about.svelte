<script lang="ts">

import { onMount } from "svelte";
import { aboutPosition, clickables } from "../store";

let aboutContainer;
let githubLink, emailLink
let profilePicContainer;

onMount(() => {

	$aboutPosition = aboutContainer.offsetTop; // Update current height for nav scrolling
	window.onresize = () => $aboutPosition = aboutContainer.offsetTop; // Update current height for nav scrolling

	clickables.update(value => [...value, githubLink, emailLink]);

	slickScroll.then((slick) => {

		slick.addOffset({
			element: profilePicContainer,
			speedY: 0.8
		});
	})
});

export let slickScroll;

</script>

<div id="content-container" class="about" bind:this={aboutContainer}>
	<div class="content-wrapper">
		<h1 class = "title">The Name's<br>Musab</h1>
		<p class = "paragraph">
			I'm a web developer from British Columbia, Canada. I love writing beautiful code and creating memorable web experiences. <br><br>I work with organizations and individuals to create taylor-made experiences designed to be beautiful, scalable, and affordable. If you like my work, you can send me an email and we can chat.
		</p>
		<div class="social-button-wrapper">
			<span class="button" bind:this={emailLink}><a href="mailto:musabhassan04@gmail.com" target="_blank" class="clickable sublink link">Email</a></span>
			<span class="button" bind:this={githubLink}><a href="https://github.com/Musab-Hassan" target="_blank" class="clickable sublink link">Github</a></span>
		</div>
	</div>
	<div class="profile-image" bind:this={profilePicContainer}>
		<img src="assets/imgs/profile-photo.jpg" alt="Musab's Sillouette" class="profile-pic">
	</div>
</div>
<div class="horizontal-flex">
	<ul class="list first">
		<li class="list-title">techstacks i use</li>
		<li>
			<div>Front-end</div>
			<div class="flex-item">
				<img src="assets/imgs/svg-icons/angular.svg" alt="angular">
				<img src="assets/imgs/svg-icons/svelte.svg" alt="svelte">
			</div>
		</li>
		<li>
			<div>Back-end</div>
			<div class="flex-item">
				<img src="assets/imgs/svg-icons/firebase.svg" alt="firebase">
				<img src="assets/imgs/svg-icons/nodejs.svg" alt="node js">
				<img src="assets/imgs/svg-icons/php.svg" alt="php">
				<img src="assets/imgs/svg-icons/mysql.svg" alt="mySQL">
			</div>
		</li>
		<li>
			<div>Design</div>
			<div class="flex-item">
				<img src="assets/imgs/svg-icons/illustrator.svg" alt="adobe illustrator">
				<img src="assets/imgs/svg-icons/xd.svg" alt="adobe xd">
			</div>
		</li>
	</ul>
	<ul class="list">
		<li class="list-title">awards</li>
		<li>1x - Awwwards Honors</li>
	</ul>
</div>


<style lang="sass">

@import "../consts.sass"
@include textStyles()

#content-container.about
	display: flex
	flex-direction: row
	justify-content: space-between
	overflow: hidden
	padding: 0 5vw
	margin-top: 40vh
	position: relative
	padding-bottom: 5vh

	.profile-image
		width: 70%
		overflow: hidden
		margin-top: -30vh

		img
			height: 80%
			width: 90%
			border-radius: 0.5vh
			object-fit: cover

	.content-wrapper
		box-sizing: border-box
		width: 50%
		height: 100%
		margin: 0 2vw
		padding-right: 4vw
		display: flex
		flex-direction: column
		justify-content: center
		margin-top: 5vh
		box-sizing: border-box
		z-index: 2

		@media only screen and (max-width: 750px)
			&
				width: 80%

				h1
					font-size: 25vw !important

		h1
			font-size: 20vh
			font-weight: 400

		.paragraph
			margin-top: 10vh
			margin-left: 13vw
			position: relative
			width: 60%
			line-height: 1.5rem

			@media only screen and (max-width: 750px)
				&
					width: 100%
					margin-left: 5vw

			&::before
				content: ""
				position: absolute
				height: 2px
				width: 11vw
				right: 110%
				top: 15%
				background-color: white

		.social-button-wrapper
			font-size: 3vh
			margin-left: 13vw
			margin-top: 4vh
			display: inline-block

			*:not(:last-child)
				margin-right: 2vw

			@media only screen and (max-width: 750px)
				&
					margin-left: 5vw


	@media only screen and (max-width: 950px)
		.right-container
			margin-top: 3vh
			justify-content: flex-start !important

		.profile-image
			display: none

.horizontal-flex
	display: flex
	flex-direction: row
	justify-content: space-between
	padding: 0 5vw
	padding-top: 2vh
	width: 100%
	box-sizing: border-box

	@media only screen and (max-width: 1080px)
		flex-direction: column
		padding: 0 8vw

	.list
		list-style-type: none
		text-align: left

		@media only screen and (max-width: 1080px)
			margin-top: 10vh

		&.first
			margin-left: 8vw

			@media only screen and (max-width: 1080px)
				margin-left: 0
		
		.flex-item
			margin-left: 5vw

		li.list-title
			letter-spacing: 1vh
			font-size: 1.5vh

		li
			font-family: $font
			text-transform: uppercase
			font-size: 2vh
			letter-spacing: 1vh
			white-space: nowrap
			padding: 2vh 0
			border-bottom: 1px solid #707070
			display: flex
			flex-direction: row
			justify-content: space-between
			align-items: center

			img
				height: 2.3vh

</style>