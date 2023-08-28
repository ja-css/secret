```
<div id="graph_{pair} class="graphs col-xs-{f}">
	<div class="graph_head">
		<div class="head">
			<div class="binance">{pair}</div>     
			<div title="Глубина рынка: объем 20 первых ордеров в стакане" id="graph_depth_{pair}" class="depth"></div>     
		</div>
		<div class="price">
			<span data-lng="price">Цена</span>
			<br />
			<span id="graph_price_{pair}" class="digit">&nbsp;</span>
		</div>
		<div class="change">
			<span data-lng="change_24_short">Изм. 24ч</span>
			<br />
			<span id="graph_price_change_{pair}" class="digit">&nbsp;</span>
		</div>
		<div class="volume">
			<span data-lng="volume_24">Объем 24ч</span>
			<br />
			<span id="graph_volume_{pair}" class="digit">&nbsp;</span>
		</div>
		<div title="Удалить график с экрана" data-pair="{pair}" class="button2 clickable delete_graph">X</div>
	</div>
	<div class="container_main">
		<div class="container__wrapper">
			<div class="container__content">
				<!-- TradingView Widget BEGIN -->
				<div class="tradingview-widget-container">
					<div id="tradingview_b858f_{pair}"></div>
						<!-- График {pair} предоставлен TradingView -->
						<script type="text/javascript">
							new TradingView.widget( 
								{
									"autosize": true, 
									"symbol": "BINANCE:{pair}",
									"interval": "{s}", 
									"timezone": "Etc/UTC", 
									"theme": "{a}", 
									"style": "1", 
									"locale": "{l}", 
									"toolbar_bg": "#f1f3f6", 
									"enable_publishing": false, 
									"hide_top_toolbar": {c}, 
									"hide_legend": {u}, 
									"hide_side_toolbar": {d}, 
									"allow_symbol_change": true, 
									"show_popup_button": true, 
									"popup_width": "1000", 
									"popup_height": "650",
									"container_id": "tradingview_b858f_{pair}"
								} 
							); 
						<\/script>
					</div><-- TradingView Widget END -->
				</div>
			</div>
		</div>
	</div>
</div>
```