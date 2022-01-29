<script lang="ts">

import { onMount } from "svelte";
import { clickables, isWorkScroll, workPosition, workScrollSpeed } from "../store";
import { ImageRenderer } from "../effects/work-slider/renderer";
import { isUnsupportedClient, letterSlide, maskSlide } from "../utils";
import { fade } from "svelte/transition";

// Slider calculations and rendering
class WorkSlider {
	currentMouseX = 0; initialMouseX = 0;
	currentPosition = 0; targetPosition = 0; initialPosition = 0;
	offsetSpeed = 4000; 
	lerpSpeed = 0.075;

    onHold = e => {
        if (currentActive != null || isMouseDown || e.target.classList.contains("button")) return;

        this.initialMouseX = e.clientX;
		this.currentMouseX = e.clientX;
        isWorkScroll.set(true);

        if (isMouseDown) {
            let style = window.getComputedStyle(listContainer);
            let matrix = new WebKitCSSMatrix(style.transform);
            this.initialPosition = matrix.m41;
        }
    }

    onRelease() {
        isWorkScroll.set(false);
    }
 
    onMouseMove = e => {
    	if (!isMouseDown) return; 
		this.currentMouseX = e.clientX;

        let diff = (this.currentMouseX - this.initialMouseX) * -1;
		this.targetPosition = this.initialPosition - (this.offsetSpeed * (diff / document.body.clientWidth));
    }

    animate = () => {
		if (currentActive === null) {
			let endPoint = listContainer.offsetWidth - document.body.clientWidth
			if (endPoint < 0) endPoint = listContainer.offsetWidth;

			// Checks for disabling overscrolling
			if (this.targetPosition > 0) this.targetPosition = 0;
			if (this.targetPosition <= (endPoint * -1)) this.targetPosition = - endPoint;
		}

        // Lerp easing
        this.currentPosition = this.lerp(this.currentPosition, this.targetPosition, this.lerpSpeed);
		
        workScrollSpeed.set(this.currentPosition - this.targetPosition); // Set Svelte Store value for the Canvas effect
        listContainer.style.transform = `translateX(${this.currentPosition}px)`;

        requestAnimationFrame(this.animate);
    }

	lerp(start, end, t) {
		return start * (1 - t) + end * t;
	}
}


let workContainer;
let container, listContainer; //Containers for Threejs meshes
let images = []; // Array of images to be passed to WebGL Shader
let workItems = []; // Array of workItems to be animated
let _viewLinks = []; // Array of clickable Links

let isMouseDown = false; // is user holding click
let currentActive = null; // Active work item in the detailsViewer viewed

let data; // JSON Work data fetched from the data.json file

// Animations for work description
let textAnimationIn = letterSlide().in;
let textAnimationOut = letterSlide().out;
let maskAnimationIn = maskSlide().in;
let maskAnimationOut = maskSlide().out;

// Svelte Store subscriptions
isWorkScroll.subscribe(val => isMouseDown = val);

const slider = new WorkSlider(); // Slider effects

const workItemsFetch = new Promise(async (resolve: (data: any[]) => void) => {
	data = await (await fetch("data.json")).json(); // Fetch work data
	resolve(data);
});

onMount(async () => {
	$workPosition = workContainer.offsetTop; // Update current height for nav scrolling
	window.onresize = () => $workPosition = workContainer.offsetTop; // Update current height for nav scrolling

	listContainer.style.transform = "translate3d(0px, 0px, 0px)";
	
	await workItemsFetch; // Wait for workItems to load

	clickables.update(values => values.concat(_viewLinks)); // Add clickables to clickables store

	if (!isUnsupportedClient()) slider.animate(); // Begin slider animations if device is not a phone
	new ImageRenderer(container, images); // ThreeJS warping effect
});



// Helper Functions
function toggleActiveItem(i) {
	currentActive = (currentActive == i) ? null : i;
	if (currentActive != null) slider.targetPosition = -(workItems[i].offsetLeft - (window.innerWidth / 4) + window.innerWidth / 10);
}

// Add clickables dynamically
function addClickable(node) {
	clickables.update(values => [...values, node]);
	return {
		destroy() {
			clickables.update(values => values.filter(item => item !== node));
		}
	}
}

// Prevent clipping of title with letters with overhand
function adjustLineHeight(node) {
	if (/[gyjqp]/g.test(node.innerText)) node.style.lineHeight = "120%";
}


</script>



<div id="content-container" class="work-click-area" style = "margin-top: 30vh;" bind:this="{workContainer}">
	<div class="content-wrapper" 
		on:mousedown|preventDefault={slider.onHold}
		on:mouseup={slider.onRelease}
		on:mouseleave={slider.onRelease}
		on:mousemove|preventDefault={slider.onMouseMove}
		bind:this={container}
		class:disabled={currentActive !== null}
	>
		<div class:mobile={isUnsupportedClient()}>
			<ul class="work-list" bind:this={listContainer} class:hold={isMouseDown}>
				<!-- Work items render here -->
				{#await workItemsFetch then items}
					{#each items as item, i}
						<li class="list-item clickable passive" 
							class:ambient="{currentActive !== i && currentActive !== null}" 
							class:active="{currentActive === i}" 
							bind:this={workItems[i]}>

							<div class="img-wrapper">
								<img bind:this={images[i]}
									on:dragstart|preventDefault 
									draggable="false" 
									src="assets/imgs/work-back/{item.id}/cover.jpg" 
									alt="{item.title} Background">
							</div>
							<div class="text-top-wrapper" class:hidden={currentActive != null || isMouseDown}>
								<p class="item-date">{item.date}</p>
							</div>
							<div class="text-wrapper" class:hidden={currentActive != null || isMouseDown}>
								<h1 class="item-title">{item.title}</h1>
								<div class="button item-link" bind:this={_viewLinks[i]} on:click={() => toggleActiveItem(i)}>view</div>
							</div>
						</li>
					{/each}
				{/await}
			</ul>
		</div>

		<!-- Active item details (When a work item is clicked) -->
		{#if currentActive !== null}
			<div class="details-container">
				<div class="wrapper">
					<div class="top-align">
						<div class="wrapper">
							<div class="index">
								<div in:maskAnimationIn out:maskAnimationOut>
									{#if (currentActive < 9)}
										{"0"+data[currentActive].id}
									{:else}
										{data[currentActive].id}
									{/if}
								</div>
							</div>
							<span class="line" transition:fade></span>
							<h6 class="caption">
								<div in:maskAnimationIn out:maskAnimationOut>{data[currentActive].details.summary}</div>
							</h6>
						</div>
					</div>
					
					<div class="mid-align">
						<h1 class="title" in:textAnimationIn={{ breakWord: false }} out:textAnimationOut use:adjustLineHeight>{data[currentActive].title}</h1>
						<div class="button" use:addClickable on:click={() => toggleActiveItem(currentActive)} in:maskAnimationIn out:maskAnimationOut>&times; close</div>
					</div>
					
					<div class="bottom-align">
						<div>
							<p class="paragraph" in:textAnimationIn out:textAnimationOut>
								{data[currentActive].details.description}
							</p>
						</div>
						<div class="links">
							{#each data[currentActive].links as link}
								<div style="position: relative">
									<a in:textAnimationIn out:textAnimationOut use:addClickable href={link.link} target="_blank" class="button no-decor">{link.text}</a>
									<div class="underline" transition:fade></div>
								</div><br>
							{/each}
							<div class="line" transition:fade></div>
						</div>
						<div class="roles">
							<div class="wrapper">
								<p class="descriptor" in:textAnimationIn out:textAnimationOut>Role</p>
								<ul>
									{#each data[currentActive].roles as role}
										<li in:textAnimationIn out:textAnimationOut>{"+ " + role}</li>
									{/each}
								</ul>
							</div>
						</div>
					</div>
				</div>
			</div>
		{/if}
	</div>
</div>


<style lang="sass">

\:global(canvas)
	position: absolute
	top: 0
	left: 0
	z-index: -1

@import "../consts.sass"
@include textStyles()

#content-container.work-click-area .content-wrapper
	display: flex
	flex-direction: column
	cursor: grab
	position: relative
	overflow: hidden

	&.disabled
		cursor: default !important

		.mobile ul.work-list
			opacity: 0

	.mobile
		width: 100%
		height: 100%
		overflow-x: auto
	
	*
		-webkit-touch-callout: none
		-webkit-user-select: none
		-moz-user-select: none
		-ms-user-select: none
		user-select: none

	.details-container
		position: absolute
		left: 0
		top: 0
		height: 100%
		width: 100%
		display: flex
		flex-direction: row
		align-item: center
		justify-content: space-between
		box-sizing: border-box
		padding: 0 14vw

		.wrapper
			text-align: left
			position: relative
			display: flex
			flex-direction: column
			justify-content: space-between

			.top-align
				.wrapper
					display: inline-flex
					flex-direction: row
					align-items: center
					justify-content: left
					position: relative

					h6.caption
						position: relative
						font-family: $font
						text-transform: uppercase
						font-weight: normal
						font-size: 1.9vh

					.index
						font: $font
						position: relative
						font-size: 2.1vh

					span.line
						width: 300%
						margin: 0 10%
						height: 1.5px
						background-color: white

			.mid-align	
				display: flex
				flex-direction: row
				align-items: center
				justify-content: space-between

				h1.title
					position: relative
					font-family: $titleFont
					font-size: 7vw
					text-transform: lowercase
					font-weight: normal
					word-wrap: break-word
					white-space: normal
					line-height: 90%

				.button
					font-size: 1.4vw
					letter-spacing: 0.1vw
					margin-top: 2vh
					text-transform: uppercase

			@media only screen and (max-width: 750px)
				.mid-align
					flex-direction: column
					justify-content: flex-start
					align-items: flex-start

					h1.title
						font-size: 16.5vw

					.button
						font-size: 2vh

			
			.bottom-align
				display: flex
				flex-direction: row
				justify-content: space-between
				align-items: center

				*
					flex-grow: 1
					flex-basis: 0

				p
					font-size: 1.4vh
					text-transform: uppercase
					width: 60%

				.roles 
					display: flex
					flex-direction: column
					align-items: center

					.descriptor
						line-height: 250%
						letter-spacing: 0.6vh
						font-family: $font
						text-transform: uppercase
						font-weight: normal
						font-size: 1.4vh


					ul 
						list-style-type: none

						li
							font-family: $font
							text-transform: uppercase
							font-weight: normal
							font-size: 1.9vh
							line-height: 160%

				.links
					position: relative
					display: flex
					flex-direction: column
					align-items: center
						
					div.line
						content: ""
						position: absolute
						top: 150%
						left: 50%
						width: 1.2px
						background-color: white
						height: 5vh

					.button
						font-size: 1.3vw
						letter-spacing: 0.2vw
						text-transform: uppercase
						text-decoration: none
						line-height: 160%

					.underline
						display: none

			@media only screen and (max-width: 750px)
				.bottom-align
					flex-direction: column
					justify-content: flex-start
					align-items: flex-start

					p
						font-size: 1.6vh !important

					.links
						margin: 2vh 0
						align-items: flex-start
						
						.button
							font-size: 2vh
							position: relative

						.underline
							display: inline-block
							position: absolute
							width: 100%
							height: 2px
							bottom: 0
							left: 0
							background-color: white

						div.line
							display: none

	ul.work-list
		margin-top: auto
		margin: auto 0
		padding: 0 5vw
		list-style-type: none
		display: flex
		flex-direction: row
		align-items: center
		height: 70vh
		min-width: min-content
		opacity: 1
		transition: opacity 0.5s ease
		-webkit-transition: opacity 0.5s ease

		&.hold
			.list-item
				height: 50vh !important

		.list-item
			display: inline-flex
			overflow: hidden
			height: 60vh
			width: 20vw
			box-sizing: border-box
			position: relative
			overflow: hidden
			z-index: 3
			margin-right: 6vw
			transition: width 0.8s cubic-bezier(0.25, 1, 0.5, 1), height 0.8s cubic-bezier(0.25, 1, 0.5, 1), margin 0.8s cubic-bezier(0.25, 1, 0.5, 1)

			*
				transition: opacity 0.3s ease
				-webkit-transition: opacity 0.3s ease

			&.active
				width: 50vw
				margin-right: 16vw
				margin-left: 10vw

				.img-wrapper
					width: 100%

			&.ambient
				height: 55vh

			.hidden
				opacity: 0

			.img-wrapper
				overflow: hidden
				height: 100%
				z-index: 1
				position: relative
				width: 85%
				box-shadow: 3px 9px 18px rgba(0, 0, 0, 0.2)
				
				img
					height: 110%
					width: 110%
					object-fit: cover
					position: absolute
					top: 50%
					left: 50%
					transform: translate(-50%, -50%)
					-webkit-transform: translate(-50%, -50%)
					opacity: 0.5

			.scroll-done
				transition: 0.8s opacity ease
				-webkit-transition: 0.8s opacity ease

			.text-top-wrapper
				position: absolute
				top: 6vh
				right: 0
				z-index: 2
				word-wrap: break-word
				white-space: normal
				text-align: right

				.item-date
					font-size: 1vw
					letter-spacing: 0.1vw
					text-transform: uppercase
					font-family: $font

			.text-wrapper
				box-sizing: border-box
				position: absolute
				bottom: 10vh
				right: 0
				text-align: right
				z-index: 2

				.button
					font-size: 1.4vw
					letter-spacing: 0.1vw
					margin-top: 2vh
					text-transform: uppercase

				.item-title
					font-family: $font
					font-weight: normal
					font-size: 2.3vw
					z-index: 0
					opacity: 1
					letter-spacing: 0.1vw
					line-height: 110%
					word-spacing: 80vw
					text-transform: lowercase
					word-wrap: break-word
					white-space: normal


				.inline-wrapper
					width: 100%
					position: relative
					margin-top: 1vh

					*
						display: inline-block

		@media only screen and (max-width: 1450px)
			.list-item
				width: 25vw

				.text-wrapper
					width: calc(40vw - 10vh)

		@media only screen and (max-width: 1110px)
			.list-item
				width: 35vw

				.text-top-wrapper
					.item-date
						font-size: 2vh

				.text-wrapper
					width: calc(55vw - 10vh)

					.item-title
						font-size: 4vw

					.item-link
						font-size: 2vh

		@media only screen and (max-width: 650px)
			.list-item
				width: 60vw

				.text-wrapper
					width: calc(70vw - 10vh)

					.item-title
						font-size: 3vh

</style>