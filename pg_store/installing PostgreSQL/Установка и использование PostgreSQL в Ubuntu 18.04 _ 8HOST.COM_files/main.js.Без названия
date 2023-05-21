jQuery(document).ready(function($) {

		var scrollEqual = $('.paralax-block').offset().top;
		var footerEquel = $('#footer').offset().top - $('.paralax-block .fixed-block').height();
		if ($(window).scrollTop() >= scrollEqual) {
			$('.paralax-block img').addClass('fixed-block');
		}
	$(window).scroll(function(event) {
		scrollEqual = $('.paralax-block').offset().top;
		footerEquel = $('#footer').offset().top - $('.paralax-block .fixed-block').height();
		// console.log(footerEquel);
		// console.log($(this).scrollTop());
		if ($(this).scrollTop() >= scrollEqual) {
			$('.paralax-block img').addClass('fixed-block');
			if (footerEquel-60 <= $(this).scrollTop()) {
				$('.fixed-block').css('top', footerEquel-$(this).scrollTop()-60+'px');
			} else {
				$('.fixed-block').css('top', 0);
			}
		} else {
			$('.paralax-block img').removeClass('fixed-block');
		}
	});

	if ($('body.plugin .footer').length) {
		$('body.plugin .footer').css('display', 'none')
	}
});