<!DOCTYPE html>
<html>
<head lang="en">

    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1">

    <title>Mining Pool</title>


    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery-timeago/1.4.0/jquery.timeago.min.js"></script>
    <script src="//cdnjs.cloudflare.com/ajax/libs/jquery-sparklines/2.1.2/jquery.sparkline.min.js"></script>
    <link href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="//netdna.bootstrapcdn.com/bootstrap/3.1.1/js/bootstrap.min.js"></script>

    <link href="//netdna.bootstrapcdn.com/font-awesome/4.1.0/css/font-awesome.min.css" rel="stylesheet">

    <link href="//fonts.googleapis.com/css?family=Inconsolata" rel="stylesheet" type="text/css">

    <script src="config.js"></script>
    <script src="custom.js"></script>

    <style>
        #coinName{
            text-transform: capitalize;
        }
        body {
            padding-top: 65px;
            padding-bottom: 80px;
            overflow-y: scroll;
        }
        .container{
            font-size: 1.1em;
        }
        #loading{
            font-size: 2em;
        }
        .stats {
            margin-bottom: 10px;
            margin-top: 5px;
        }
        .stats:last-child{
            width: auto;
        }
        .stats > h3 > i{
            font-size: 0.80em;
            width: 21px;
        }
        .stats > div{
            padding: 5px 0;
        }
        .stats > div > .fa {
            width: 25px;
        }
        .stats > div > span:first-of-type{
            font-weight: bold;
        }
        #stats_updated{
            opacity: 0;
            float: right;
            margin-left: 30px;
            color: #e8e8e8;
            line-height: 47px;
            font-size: 0.9em;
        }

        footer{
            position: fixed;
            bottom: 0;
            width: 100%;
            background-color: #f5f5f5;
        }

        footer > div{
            margin: 10px auto;
            text-align: center;
        }

    </style>
    
    <link href="custom.css" rel="stylesheet">
<script id="chatBroEmbedCode">
/* Chatbro Widget Embed Code Start */
function ChatbroLoader(chats,async){async=!1!==async;var params={embedChatsParameters:chats instanceof Array?chats:[chats],lang:navigator.language||navigator.userLanguage,needLoadCode:'undefined'==typeof Chatbro,embedParamsVersion:localStorage.embedParamsVersion,chatbroScriptVersion:localStorage.chatbroScriptVersion},xhr=new XMLHttpRequest;xhr.withCredentials=!0,xhr.onload=function(){eval(xhr.responseText)},xhr.onerror=function(){console.error('Chatbro loading error')},xhr.open('GET','//www.chatbro.com/embed.js?'+btoa(unescape(encodeURIComponent(JSON.stringify(params)))),async),xhr.send()}
/* Chatbro Widget Embed Code End */
ChatbroLoader({encodedChatId: '3XxQ'});
</script></head>
<body>
<script>


    var docCookies = {
        getItem: function (sKey) {
            return decodeURIComponent(document.cookie.replace(new RegExp("(?:(?:^|.*;)\\s*" + encodeURIComponent(sKey).replace(/[\-\.\+\*]/g, "\\$&") + "\\s*\\=\\s*([^;]*).*$)|^.*$"), "$1")) || null;
        },
        setItem: function (sKey, sValue, vEnd, sPath, sDomain, bSecure) {
            if (!sKey || /^(?:expires|max\-age|path|domain|secure)$/i.test(sKey)) { return false; }
            var sExpires = "";
            if (vEnd) {
                switch (vEnd.constructor) {
                    case Number:
                        sExpires = vEnd === Infinity ? "; expires=Fri, 31 Dec 9999 23:59:59 GMT" : "; max-age=" + vEnd;
                        break;
                    case String:
                        sExpires = "; expires=" + vEnd;
                        break;
                    case Date:
                        sExpires = "; expires=" + vEnd.toUTCString();
                        break;
                }
            }
            document.cookie = encodeURIComponent(sKey) + "=" + encodeURIComponent(sValue) + sExpires + (sDomain ? "; domain=" + sDomain : "") + (sPath ? "; path=" + sPath : "") + (bSecure ? "; secure" : "");
            return true;
        },
        removeItem: function (sKey, sPath, sDomain) {
            if (!sKey || !this.hasItem(sKey)) { return false; }
            document.cookie = encodeURIComponent(sKey) + "=; expires=Thu, 01 Jan 1970 00:00:00 GMT" + ( sDomain ? "; domain=" + sDomain : "") + ( sPath ? "; path=" + sPath : "");
            return true;
        },
        hasItem: function (sKey) {
            return (new RegExp("(?:^|;\\s*)" + encodeURIComponent(sKey).replace(/[\-\.\+\*]/g, "\\$&") + "\\s*\\=")).test(document.cookie);
        }
    };

    function getTransactionUrl(id) {
        return transactionExplorer.replace('{symbol}', lastStats.config.symbol.toLowerCase()).replace('{id}', id);
    }
    
    $.fn.update = function(txt){
        var el = this[0];
        if (el.textContent !== txt)
            el.textContent = txt;
        return this;
    };

    function updateTextClasses(className, text){
        var els = document.getElementsByClassName(className);
        for (var i = 0; i < els.length; i++){
            var el = els[i];
            if (el.textContent !== text)
                el.textContent = text;
        }
    }

    function updateText(elementId, text){
        var el = document.getElementById(elementId);
        if (el.textContent !== text){
            el.textContent = text;
        }
        return el;
    }

    function updateTextLinkable(elementId, text){
        var el = document.getElementById(elementId);
        if (el.innerHTML !== text){
            el.innerHTML = text;
        }
        return el;
    }

    var currentPage;
    var lastStats;


    function getReadableHashRateString(hashrate){
        var i = 0;
        var byteUnits = [' H', ' KH', ' MH', ' GH', ' TH', ' PH' ];
        while (hashrate > 1000){
            hashrate = hashrate / 1000;
            i++;
        }
        return hashrate.toFixed(2) + byteUnits[i];
    }

    function formatBlockLink(hash){
        return '<a href="' + getBlockchainUrl(hash) + '">' + hash + '</a>';
    }

    function getReadableCoins(coins, digits, withoutSymbol){
        var amount = (parseInt(coins || 0) / lastStats.config.coinUnits).toFixed(digits || lastStats.config.coinUnits.toString().length - 1);
        return amount + (withoutSymbol ? '' : (' ' + lastStats.config.symbol));
    }

    function formatDate(time){
        if (!time) return '';
        return new Date(parseInt(time) * 1000).toLocaleString();
    }

    function formatPaymentLink(hash){
        return '<a href="' + getTransactionUrl(hash) + '">' + hash + '</a>';
    }

    function getPaymentRowElement(payment, jsonString){

        var row = document.createElement('tr');
        row.setAttribute('data-json', jsonString);
        row.setAttribute('data-time', payment.time);
        row.setAttribute('id', 'paymentRow' + payment.time);

        row.innerHTML = getPaymentCells(payment);

        return row;
    }


    function parsePayment(time, serializedPayment){
        var parts = serializedPayment.split(':');
        return {
            time: parseInt(time),
            hash: parts[0],
            amount: parts[1],
            fee: parts[2],
            mixin: parts[3],
            recipients: parts[4]
        };
    }

    function renderPayments(paymentsResults){

        var $paymentsRows = $('#payments_rows');

        for (var i = 0; i < paymentsResults.length; i += 2){

            var payment = parsePayment(paymentsResults[i + 1], paymentsResults[i]);

            var paymentJson = JSON.stringify(payment);

            var existingRow = document.getElementById('paymentRow' + payment.time);

            if (existingRow && existingRow.getAttribute('data-json') !== paymentJson){
                $(existingRow).replaceWith(getPaymentRowElement(payment, paymentJson));
            }
            else if (!existingRow){

                var paymentElement = getPaymentRowElement(payment, paymentJson);

                var inserted = false;
                var rows = $paymentsRows.children().get();
                for (var f = 0; f < rows.length; f++) {
                    var pTime = parseInt(rows[f].getAttribute('data-time'));
                    if (pTime < payment.time){
                        inserted = true;
                        $(rows[f]).before(paymentElement);
                        break;
                    }
                }
                if (!inserted)
                    $paymentsRows.append(paymentElement);
            }

        }
    }

    function pulseLiveUpdate(){
        var stats_update = document.getElementById('stats_updated');
        stats_update.style.transition = 'opacity 100ms ease-out';
        stats_update.style.opacity = 1;
        setTimeout(function(){
            stats_update.style.transition = 'opacity 7000ms linear';
            stats_update.style.opacity = 0;
        }, 500);
    }

    window.onhashchange = function(){
        routePage();
    };


    function fetchLiveStats() {
        $.ajax({
            url: api + '/live_stats',
            dataType: 'json',
            cache: 'false'
        }).done(function(data){
            pulseLiveUpdate();
            lastStats = data;
            updateIndex();
            currentPage.update();
        }).always(function () {
            fetchLiveStats();
        });
    }

    function floatToString(float) {
        return float.toFixed(6).replace(/[0\.]+$/, '');
    }


    var xhrPageLoading;
    function routePage(loadedCallback) {

        if (currentPage) currentPage.destroy();
        $('#page').html('');
        $('#loading').show();

        if (xhrPageLoading)
            xhrPageLoading.abort();

        $('.hot_link').parent().removeClass('active');
        var $link = $('a.hot_link[href="' + (window.location.hash || '#') + '"]');

        $link.parent().addClass('active');
        var page = $link.data('page');

        xhrPageLoading = $.ajax({
            url: 'pages/' + page,
            cache: false,
            success: function (data) {
                $('#loading').hide();
                $('#page').show().html(data);
                currentPage.init();
                currentPage.update();
                if (loadedCallback) loadedCallback();
            }
        });
    }

    function updateIndex(){
        updateText('coinName', lastStats.config.coin);
        var title = $(".navbar-brand").text();
        $("title").text(title.charAt(0).toUpperCase() + title.slice(1));
        updateText('poolVersion', lastStats.config.version);
    }

    function getBlockchainUrl(id) {
        return blockchainExplorer.replace('{symbol}', lastStats.config.symbol.toLowerCase()).replace('{id}', id);
    }

    $(function(){
		$("head").append("<link rel='stylesheet' href=" + themeCss + ">");
		
        $.get(api + '/stats', function(data){
            lastStats = data;
            updateIndex();
            routePage(fetchLiveStats);
        });
    });

    // Blockexplorer functions
    urlParam = function(name){
        var results = new RegExp('[\?&]' + name + '=([^&#]*)').exec(window.location.href);
        if (results==null){
           return null;
        }
        else{
           return results[1] || 0;
        }
    }
</script>

<div class="navbar navbar-inverse navbar-fixed-top" role="navigation">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand active" href="#"><span id="coinName"></span> Mining Pool</a>
        </div>
        <div class="collapse navbar-collapse">
            <ul class="nav navbar-nav" data-toggle="collapse" data-target=".navbar-collapse">

                <li style="display:none;" class="active"><a class="hot_link" data-page="home.html" href="#">
                    <i class="fa fa-home"></i> Home
                </a></li>

                <li><a class="hot_link" data-page="getting_started.html" href="#getting_started">
                    <i class="fa fa-rocket"></i> Getting Started
                </a></li>

                <li><a class="hot_link" data-page="pool_blocks.html" href="#pool_blocks">
                    <i class="fa fa-cubes"></i> Pool Blocks
                </a></li>

                <li><a class="hot_link" data-page="payments.html" href="#payments">
                    <i class="fa fa-paper-plane-o"></i> Payments
                </a></li>

<!-- //-->
                <li><a class="hot_link" data-page="blockchain_blocks.html" href="#blockchain_blocks">
                    <i class="fa fa-cubes"></i> Blockchain
                </a></li>

                <li style="display:none;"><a class="hot_link" data-page="blockchain_block.html" href="#blockchain_block">
                </a></li>

                <li style="display:none;"><a class="hot_link" data-page="blockchain_transaction.html" href="#blockchain_transaction">
                </a></li>
<!-- //-->

                <li><a class="hot_link" data-page="network.html" href="#network">
                    <i class="fa fa-chain"></i> Network
                </a></li>

                <li><a class="hot_link" data-page="support.html" href="#support">
                    <i class="fa fa-comments"></i> Support
                </a></li>



            </ul>
            <div id="stats_updated">Stats Updated &nbsp;<i class="fa fa-bolt"></i></div>
        </div>

    </div>
</div>

<div class="container">

    <div id="page"></div>

    <p id="loading" class="text-center"><i class="fa fa-circle-o-notch fa-spin"></i></p>

</div>

<footer>
    <div class="text-muted">
        Powered by <a target="_blank" href="https://github.com/forknote/cryptonote-forknote-pool"><i class="fa fa-github"></i> cryptonote-forknote-pool</a>
        <span id="poolVersion"></span>
        open sourced under the <a href="https://www.gnu.org/licenses/gpl-2.0.html">GPL</a>
    </div>
</footer>


</body>
</html>
