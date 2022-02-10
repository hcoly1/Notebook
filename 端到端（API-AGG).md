# 端到端（API-AGG)

## BFF层：



## 列表查询

- Streaming模式，分两批返回
- Soa模式

根据报文进行的推测：

1、列表查询也返回多运价，因为底层返回的列表查询的response中包含多个offer，但是因为列表查询只显示最低的，所有存在过滤的逻辑?

### agg端响应报文分析：

```txt
- ResponseBody
	- Itinerary 行程组合
		- transportTypeGroup 行程组合类型，所有的transportType的位或 操作 如：3-flight+tran
		- transportSegment 行程、段索引
			- journeyNo 行程号，从1开始
			- segmentNo 每程内的段号，从1开始
			- transportRef 交通工具信息的索引号
			- mainSegmentInd 是否行程的主航段
			- tag 扩展节点 key-value
        - offer 出价信息
        	- transportFareMapping 与transportRef一一对应
        		- transportRef 航班、火车、bus等的详情索引号
        		- paxSeat 乘客类型对应的舱等舱位信息
        			- ageTypeGroup 对应的乘客类型组，可表示多个乘客类型，1：adult 2：child 4：infant
        			- seatCount 余票信息，国内机票的婴儿始终10
        			- productDetailIndex 与productDetailList的下标关联，从0开始
        			- flightSeat 机票舱位及相关信息
        				- puSequence 运价的puSequence
        				- fcSequence 运价的fcSequence
        				- cabinCode 机票舱等 Y：经济舱 W： 超级经济舱 C：公务舱 F：头等舱
        				- rBD 子舱位
        				- marketingCabinCode 舱等区域code：Y、S、C、F
        				- SeatSource 来源：0：正常av 1：库存 2：martain 3：境外舱位 4：定额 5：od
        				- stockId 库存id 库存类产品减库存的依据
        				- mileageCredit 航班舱位可累计里程
        				- rBDDisclosure 国内特别舱位信息
        					- name 舱位名称
        					- shortname 舱位简称
        					- description 舱位描述
        			- tranSeat 火车座位及相关信息
        				- seatTypeName 座位类型名称
        	- productDetail 行程组合的价格单元列表
        		- transportProductType 交通工具的价格类型 1：flightProductType 2：train 3：bus
        		- productRef flightProduct、trainProduct、busProduct对应的索引号
        		- compatiblePenaltyKey 退改签反查key 对应国际的penaltykey，国内的rcKe
        	- offerToken 大交通价格反查token agg内部解析使用，对外不保证一致
        	- productCombinationType 产品组合类型 包括单程直飞的部分信息
        	- offerPackage 绑定在offer维度的辅营产品
        		- extensionField gv产品说明 muc
        			- productActionMode 产品处理方式，按位与 1：拦截 2：注册
        			- productRemark 产品标识说明
        		- extramarketingField 其他营销信息 兼容引擎和agg适配的信息 key-value
        		- bundleItem 附加产品列表
        			- bundleType 产品类型，目前支持的取值类型giftProduct cashback couponproduct flightxcomposition hotel ancillaries  memberprivilege PriceReduction BrandAttribute
        			- productRef 绑定信息的索引
        			- serviceAssociation 该产品所属的航段信息，为空表示属于整个行程
        				- journeyNo 绑定的行程号 从1开始
        				- segmentNo 某一程内的段号 从0开始
        			- ageTypeGroup 适用的乘客类型组，可表示多个乘客类型
                    - productDetailIndex 
                    - tag 绑定附加属性 key-value
            - baggageRef 行李信息
               	- transportRef 航班、火车、bus等的详情索引号
               	- paxBaggage
              		- ageTypeGroup 适用的乘客类型组
               		- baggageWay 行李额类型
               			- baggageTypeRef 关联baggageType.refNum
               			- type 0: 托运 1：手提
               		- baggageSpecificDesc 行李描述 多语言
            - baggageAncillary 增值行李：区分打包和加购方式，且每种方式可能有不同的规格
            	- bundleItem 附加产品列表 同上 offerpackage.bundleItem
            	- baggageRef 行李信息 同上 offer.packageref
            	- orderWay 1：打包 2：加购
            - priceAttributeID 国际机票价格属性ID集合
            - tag 扩展节点 key-value
            - recommendedType  推荐运价类型 0：普通运价 1：推荐运价 2：普通+推荐（与前端展示相关）
            - transportHotelToken 机酒token 
            - priority 优先级 越小越高
            - journeyAttachment 与行程相关的信息，元素最多与行程数一样
            	- journeyNo 行程号，从1开始
            	- productCombinationType 国内的产品组合类型， RT/MT时有值
        - flightManType 0: 不是飞人 1：普通飞 2：超级飞
        - interchange 换乘信息
        	- journeyNo 行程号，从1开始
        	- segmentNo 每一程内段号，从1开始
        	- position 换乘信息是在段前还是在段后 1：前 2：后
        	- interchangeRoute 换乘的格式化信息，List表示不同的换乘路线
        		- interchangeStep 多种交通工具的有序组合
        			- vehicleType 1： tran 2：taxi 3：公交巴士 4： 地铁 5：自驾 6：长途汽车
        			- durationMinutes 时间 单位分 
        			- distance 距离 单位公里
        			- price 价格
        			- remark 提示信息
        	- textRemarkRef 换乘的非格式化信息，索引号
        	- checkInRemarkRef checkIn的描述信息，索引号
        - tag 扩展节点
        - priority 优先级 越小越高   
       
	- metaData  
		- saleCurrency 销售币种
	- compatibleDetailToken 国际Agg的SearchCriteriaToken，调国际Agg的DetailSearch接口时使用; 国内Agg的ResponseToken，调国内Agg的GetProductDetails接口时使用;
	- hotelLoadToken 机酒缓存Token，调机酒服务时使用
	- searchCriteriaToken 对应了请求参数SearchCriteriaType, 调用本服务的DetailSearch接口时使用
```

```txt
	- dataList 数据列表
    	- transport 交通信息ref
    		- refNum 索引号
    		- type 交通工具类型 1：flight 2：train 4：bus
    		- transportNo 航班号，火车车次等
    		- departDateTime 出发时间 yyyy-MM-dd HH:mm:ss
    		- departPoint 出发点信息
    			- cityId 城市id
    			- airportCode 机场三字码
    			- terminalName 航站楼
    			- stationName 站点名
    		- arriveDateTime 到达时间 格式如上departDateTime
    		- arrivePoint 到达点信息 格式如上departPoint
    		- durationMinutes 时间
    		- tag 扩展节点 key-value
    		- flight flight类型扩展信息
    			- marketingCarrierCode 展示航司
    			- operatingCarrierCode 承运航司
    			- operatingFlightNo 承运航班号
    			- aircraftCode 机型
    			- virtualFlightSupplier 虚拟航班供应商
    			- flightReferencePrice 航班相关的其他参考价格信息
    				- standardPrice 标准价（公布运价）舱位的政策信息列表
    					- cabinCode 舱等 Y W C F
    					- rbd 标准价对应的子舱
    					- publishPrice 标准价
    				- upgradeCFReferencePrice 升舱参考价 采集航班下F舱等的最低价或者全价,用于给前端展示升舱产品优势价差；优势价差 = 升舱参考价-客人购买经济舱价格-升舱费用
    			- stop 经停
    				- airportCode 机场三字码
    				- durationMinutes 时间
    				- arriveDateTime 到达时间
    			- throughFlight 通程信息 （联程？）
    				- transpRef 所属通程航班的索引号
    				- extra 通程航班数据
    			- punctuality 准点信息
    				- arriveHistoryPunctuality 到达的历史准点率 带%
    				- estimateDepartTime 预计出发时间 HH：mm 
    				- estimateFlightStatus 预计航班状态 0：计划 1：延误
    				- estimateArriveDelayMinMinutes 预测最少延误时间
    				- estimateArriveDelayMaxMinutes 预测最大延误时间
    				- delayReason 预测延误原因名称列表 key-value
    			- virtualFlightType 虚拟航班类型 0：飞机 1：火车 2：汽车 3：水上飞机
    	- flightProduct
        	- refNum 索引号
            - validatingCarrier 出票航司
            - bookingChannel 查询channel
            - paxProduct 根据乘客类型划分的航班价格信息
            	- eligiblity 乘客资质 底层返回的原始值
            	- ageType 乘客类型 1：adult 2：child 4：infant
            	- saleCurrencyPriceDetail 销售币种的价格信息
            		- salePrice 售价
            			- value 金额
            			- payee 收款方 0：携程代收 1： 航司收款
            		- tax 税费 结构同上 salePrice
            		- publishPrice 票面价（公布价）
            		- discount 折扣
            			- amount 折扣金额 结构同上 salePrice
            			- mode 折扣模式 0：无效 1：虚拟折扣 2： 三方折扣
            			- showType 折扣展示类型 0：无效 1：标签 2： 已减
            			- discountLabel 个性化标签
            				- label
            			- rebateType 国际查询列表反查阶段，将rebateType记录为让利
            		- rate 折扣率（售价/舱等全价）
            		- prePcCodePrice 国内航司 优惠前价格
            		- serviceFee 服务费明细列表 
            			- category 服务费种类 1：携程订位服务
            			- amount 服务费 结构同上salePrice
            		- flightHotelDiscountAmount 机酒专享政策优惠金额（与外放最低价的价差）
            	- settlementCurrencyPriceDetail 结算币种的价格信息
            		- currency 币种
            		- salePrice 售价 结构同上 saleCurrencyPriceDetail.salePrice
            		- tax 税费 结构同上 saleCurrencyPriceDetail.tax
          	 		- publishPrice 票面价（公布价）
          	 		- accountPrice 结算底价 不含后返
          	 		- netPrice 结算底价 包含后返
          	 		- exchangeRate 结算币种到销售币种的汇率
          	 		- serviceFee 服务费明细列表 结构同上 saleCurrencyPriceDetail.serviceFee
          	 	- rebate 后返信息
          	 		- type 类型
          	 		- amount 金额
          	 		- currency 币种
          	 		- rebatePolicyID
          	 		- actualAmount
          	 	- ticketRemark 出票备注
          	 	- paxRestriction 乘客限制信息
          	 		- ageLimitType 年龄限制类型 0： all passengers 1： one passenger at least
          			- ageLimitRange 年龄限制，部分特殊运价包含多个范围
                    	- min 下限
                    	- max 上限
                    - nationalityAllow 国籍白名单
                    - nationalityBlock 国籍黑名单
                - textRemarkRef 备注信息索引
                - refundChange 退改签简体信息
                	- refundType 是否可退票 1：不允许 2：允许 3：有条件
                	- changeType 是否可更改 1：不允许 2：允许 3：有条件
                   	- endorseType 是否可签转 1：不允许 2：允许 3：有条件
                   	- minRefundFee
                   		- value
                   	- minChangeFee 结构同上
                - tag 扩展节点
            - productType 产品类型
            - invoiceType 提供行程单或发票选项：1: 发票, 2: 行程单, 4: 行+差, 8: 境外电子凭证
            - restriction 限制类信息
            	- languageOfGovernmentIssuedPhotoID 使用本地证件预订票的语言
            	- paymentLimit 支付限制
            		- prepayType 支付类型， “拿去花”等
            		- creditCardPaymentLimitRef 信用卡支付限制的信息索引
            		- paymentDiscountRefType 支付卡让利信息
            			- paymentDiscountRef 折扣索引列表
            			- creditCardType 支付卡类型
            	- certificateTypeAllow 证件类型白名单，空表示不限制
            	- certificateTypeBlock 证件类型黑名单
            	- iDCardNos 身份证号限制,如 "31,3
            	- ctripMemberships 携程会员,如 "GOLD,DIAMOND"
            	- ctripAgreementIDAllow 航司或携程的会员定向投放的用户UID(定向Code), 白名单
                - ctripAgreementIDBlock 航司或携程的会员定向投放的用户UID(定向Code)，黑名单
                - carrierAgreementID 直联政策产品类型标识(定向Code)
                - groupTicket 1: 团队票，2:多人直减
                - identityLimit 身份限制信息，0:无限制; 1:Student; 2:军警残
                - paxCountRange 人数限制 (婴儿不算人头)
                	- min 下限
                	- max 上限
                - PaxNumLimitType 国内票人数限制类型，(0:儿童不计入总人数,1:儿童计入总人数)
            - productCategory  
            	None = 0; Exclusive = 1; Cloud = 2; Prioritizing = 4; LowestPrice = 8; FlagShipStore = 16;
            - subProductCategory 
            	None = 0; Platforms = 1; PlatformsPrivate = 2; OverseasPrivate = 3; Overseas = 4; Owner = 5; Pseat = 6; RTSeat = 7; CSD = 8; CSDPrivate = 9; Airline = 10; LowestPrice = 11; KWPrivate = 12; CSDPublish = 13; OSeat = 14;
            - cacheRef
            - ticketingTimeLimit
            	- ticketLimitType 出票保护时限类型 1：订单生成出票单后 2：航班起飞时间前
            	- limitMinutes 出票保护时限 单位分
            	- averageMinutes 平均出票时长
            - fareRef 运价fare信息
            	- puSequence 
            	- fcSequence
            	- ageTypeGroup 对应的乘客类型组
            	- fareInfoRef fareInfo对应的索引号
            - carrierTicketingDeadline 航司最晚出票时间
            - eligibilityRemarkRef 资质描述信息的refID
            - recommendPrdType 国际机票标签类信息
            - flightSupplier 供应商信息
            	- agencyID 票台ID
            	- officialName 供应商名称 iata号码的信息放在IATANumber节点
            	- IATANumber IATA号
            	- agentDisclosure 票台披露信息，如代理资质：中国国航授权代理
            - carrierPriceShow 按航司显示舱位，票面价
            	- rBD 子舱
            	- price 价格 销售币种
            - grabTicket 抢票
            	- grabTicketInd 是否是抢票
            	- successRateScope 区间成功率
            		- name AVG、24H、72H、720H、PQ_AVG、PQ_24H、PQ_72H、PQ_720H、PQ_HIGH、PQ_LOW等
            		- rate 成功率
            - membershipRegisterType 会员注册 1：联系人 2：单个乘机人 3：多个乘机人
            - pcCode 航司直销pc码
            - marketingCode 营销活动代码
            - tag 扩展节点
        - fareInfo fare信息
        	- refNum 索引号
        	- fareID 运价ID
        	- fareBasis 运价基础
        	- fareSource 运价来源，目前来看0GDS下载，1文件，3公布，5协议，6商旅，7FBR，9境外一国，10境外跨国，11美加运价，12特殊运价
        	- ownerCarrier 运价发布航司
			- fareType 运价类型
			- accountCode
			- ticketDesignator 运价的ticketCode 
       		- tourCode
       		- eligibility 运价资质
       		- tariffNo 运价区域
       		- fareRule 运价规则
       		- fareTypeCode 运价FTC
       		- fareClass
       	- baggage 行李额信息ref
       		- refNum
       		- baggaDetail
       			- amount -1表示不限制
       			- unit pc kg
       			- type piece weight
       		- extensionField
       			- containFreeBaggage 是否包含免费行李
       		- baggageSize
       			- sizeLimitType 尺寸限制类型 1：三边之和 2：长*宽*高 3：三边之和&三边各长宽高（&符合分隔）
       			- size 尺寸大小 如：180或10*12*14或180&10*12*14
       			- unit 尺寸单位 如：cm
       	- marketingProduct 辅营产品信息ref
       		- serviceDefinition x产品
       			- refNum 索引号
       			- servicePolicy 绑定service的政策
       				- policyID 政策ID
       				- marketingType 营销类型(1赠送,2立减,3定价销售)
       				- marketingAmount 营销金额
       				- activeCode 活动code
       				- retain 配置模块是否挽留,true 表示挽留，false 表示不挽留
       				- sceneCode BI场景化推荐Code
       			- serviceDetail service的详细定义
       				- productType 
       				国内目前支持的取值有：CouponProduct、VIPLounge、Specialty、InsuranceProduct、SecurityChannel、PickUp、PostCard、ServicePackage、MemberInterest、VirtualProduct、DiningCoupon、Metro、Railway 
       				国际支持的取值：Pickup,VIPLounge,FlightBoat, FlightCar,GeneralCoupon,ServicePackage,MemberInterest,BaggageAncillary, TransferService
       				- subProductType 子产品类型
       				- productID 产品ID
       				- publishPrice 票面价
       				- salePrice 售价
       				- marketPrice 市场价
       				- name 名称
       				- shortName 简称
       				- count 售卖数量
       				- cancellable 是否可取消
       				- productCode 产品代码
       				- extension   key-value
       				各个产品不通用的附加属性，如 通用券-additionProductInfoID/展示类型 巴士-邻近城市ID 接送机-接送类型/验证码/车的类型
       			- serviceDefinitionRefs 索引到其他ServiceDefinition，可表示绑定关系
       			- serviceToken 透传X前置接口的policyToken/对应国内X产品倒查key
       		- coupon 优惠券
       			- refNum 
       			- type 优惠券类型
       			- couponCode 优惠券券码
       			- discountAmount 折扣金额 折扣金额，指购买此打包产品时总价的优惠，最终的销售价为 (机票价格+Coupon.SalePrice-Coupon.DiscountAmount)
       			- salePrice 券销售价格
       		- gift 礼盒
       			- refNum 
       			- giftID 礼盒ID，数据库中ID
       			- iconName 图标名字
       			- quantity 礼盒数量
       			- showType 展示形式(0 普通礼盒 礼盒展示在价格；1：航班礼盒,2：航线礼盒)
       			- type 礼盒类型 ，0:未指定，此时ServiceList中包含多个服务类型，1：积分，2：保险，3：抵用券，4：其他，5：经深飞，6：休息室，8：优惠券，10：三亚航班，11：酒店住宿，12：行李直挂，13：机场餐食，来自DB
       			- marketingCode 营销code
       			- giftService 
       		    中转服务，此时GIftType=0。中转环节会用，用于基础类服务标识——行李直挂、一票到底、航班变无忧、行李代转运，一个礼盒中可以勾选多个类型
       		    	- serviceID 服务ID
       		    	- serviceName 服务名
       		    	- serviceType 服务类型，1：基础类服务，2：增值类服务
       		    	- checked 礼盒界面上是否勾选当前服务,如中转的行李直挂、放心出行等，前端会根据此信息来展示不同的文本, 0:false,1:true
       		- brandAttribute 权益ref
       			- refNum 
       			- type 权益类型  1: 品牌权益； 2:普通权益
       			- brandName 品牌名字，没有品牌运价传默认值
       			- brandTier 品牌等级，没有品牌运价传默认值
       			- serviceDetail 具体权益信息列表 key-value
       			- serviceDetailID 品牌运价对应的权益属性ID
                - ctripBrandTier 携程品牌运价等级
       		- cashback 返现
       			- refNum 
       			- type 类型, 1: 普通返现, 2: 溢价返现（高返政策返现), 3: 任我游礼品卡, 4: 任我行礼品卡, 5:里程。国际机票的返现金类型是2
       			- amount 金额
       		- ancillaries 增值服务
       			- refNum 
       			- checkInServiceStatus 值机服务属性(0:不支持在线购买,1:支持在线购买,2：必须在线购买)
       		- memberPrivilege 定投
       			- refNum
       			- promotionID 定投ID
       			- forUser 使用人群范围
       		- priceReduction 立减
       			- refNum
       			- promotionID 立减ID
       			- forUser 使用人群范围
       			- discountAmount 立减金额
       			- couponCode 券规则code
       			-selectedGroups 展示人群
       	- cache 缓存响应ref
        	- refNum 
        	- ageInMillisecond
        	- traceID
        - creditCardPaymentLimit 信用卡支付信息ref
        	- refNum
        	- creditCardType 可用的信用卡组织
        	- adultFee 成人卡费
        		- currency 对应币种
        		- minAmount 最小费用
        			- value 金额
        			- payee 收款方, 0:表示携程代收代付，1:表示航司收款
        		- maxAmount 最大费用 结构同上 最小费用
        	- childFee 儿童卡费 结构同上 成人卡费
        	- infantFee 婴儿卡费 结构同上 成人卡费
        - textRemark 文本引用信息
        	- refNum
        	- text 根据请求字段Language展示文本
        - paymentDiscount 让利信息
        	- refNum
        	- discounttotalFee 让利总费用=所有乘客让利之和
        	- currency 币种
        	- discountSettle 让利支付方式
        	- bankCode 发卡行
        	- cardLevel 让利卡等级
        	- discountDetail 让利明细
        		- discountType 让利类型
        		- discountFee 让利费用  				
```

```txt
- recommendItinerary 推荐信息
	- type 推荐位类型
	- redirectItinerary 推荐位简体结构，重新列表查询
		- redirectSegment 推荐简体结构中每一段的信息
			- journeyNo 程号，从1开始
			- segmentNo 每程内的段号，从1开始
			- transportType 交通工具类型 1-flight; 2-train; 4-bus
			- departCityID 出发城市ID
			- departSite 出发机场或铁路或巴士起点站
			- arriveCityID 到达城市ID
			- arriveSite 到达机场或铁路或巴士终点站
			- departDateTime 出发时间 format: yyyy-MM-dd HH:mm:ss
			- arriveDateTime 到达时间 format: yyyy-MM-dd HH:mm:ss
			- transportNo 航班号 火车车次号等
		- salePrice 售价
		- interchange 换乘信息
        	- journeyNo 行程号，从1开始
        	- segmentNo 每一程内段号，从1开始
        	- position 换乘信息是在段前还是在段后 1：前 2：后
        	- interchangeRoute 换乘的格式化信息，List表示不同的换乘路线
        		- interchangeStep 多种交通工具的有序组合
        			- vehicleType 1： tran 2：taxi 3：公交巴士 4： 地铁 5：自驾 6：长途汽车
        			- durationMinutes 时间 单位分 
        			- distance 距离 单位公里
        			- price 价格
        			- remark 提示信息
        	- textRemarkRef 换乘的非格式化信息，索引号
        	- checkInRemarkRef checkIn的描述信息，索引号
```

### api端响应报文分析：

```txt
- flightProductList 机票产品信息   数据来源：agg返回的查询结果
	- segmentList 行程信息        数据来源：OfferDTO, segmentNo, FlightListSearchContext                      
		- segmentNo 航程号        OfferDTO.TransportSegmentDTO.JourneyNo
		- departCity 出发城市     OfferDTO.TransportSegmentDTO.TransportDTO.PointDTO.cityId
			- code 城市三字码 
    		- name 城市名
    		- id 城市编号
    		- region 城市所属地区（CN：中国大陆，HK：中国香港，JP:日本等）
    		- timeZone 时区
    		- isInternational 是否为国际城市
    		- enName 城市英文名称
		- arriveCity 到达城市 结构同上
		- departDateTime 出发时间    OfferDTO.TransportSegmentDTO.TransportDTO.departDateTime
		- arriveDateTime 到达时间    OfferDTO.TransportSegmentDTO.TransportDTO.arriveDateTime
		- days 总天数                根据上述出发/到达时间计算
		- duration 飞行时间          根据上述出发/到达时间计算
			- hour 小时
			- minute 分钟
		- mainCabinClass 主舱        OfferDTO.TransportFareMappingDTO.PaxSeatDTO.FlightSeatDTO.cabinCode
		- flightList 航段信息         TransportFareMappingDTO.TransportSegmentDTO.TransportDTO.FlightDTO
			- sequenceNo 航段号，从1开始	transportSegment.getSegmentNo()
			- cabinClass 舱位				TransportFareMappingDTO.PaxSeatDTO.FlightSeatDTO.cabinCode
			- subClass 子舱               TransportFareMappingDTO.PaxSeatDTO.FlightSeatDTO.rBD
			- flightNo 航班号				transportDTO.getTransportNo()
			- airline 航司信息             flightDTO.getMarketingCarrierCode()
				- code 航司三字码
				- name 航司名称
				- lcc 是否廉航
				- alliance 航司联盟
			- shareFlightNo 共享航班号                   flightDTO.getOperatingFlightNo()
			- shareAirline 共享航司信息 结构同上 airline   flightDTO.getOperatingCarrierCode()
			- days 总天数                                transportDTO.getDepartDateTime()/getArriveDateTime()
			- duration 飞行时间                          transportDTO.getDurationMinutes()
				- hour 小时
				- minute 分钟
			- aircraft 机型信息                          flightDTO.getAircraftCode()
				- craftType 机型
	        	- name 机型全称
        		- widthLevel 宽度等级
        		- minSeats 最少座位数
        		- maxSeats 最大座位数
        		- level 类型（Normal、Big）
    	    	- widthLevelName (Normal、Big的多语言)、
	        	- lowestPrice 最低价
	        - departCity 出发城市 结构同上               transportDTO.getDepartPoint().getCityID()
	        - arriveCity 到达城市 结构同上               transportDTO.getArrivePoint().getCityID()
	        - departAirport 出发机场 		PointDTO.getAirportCode()/getTerminalName()/getDepartDateTime()
	        	- code 机场三字码
	        	- name 机场名称
	        	- terminal 航站楼
	        - arriveAirport 到达机场 结构同上 departAirport
	        - departDateTime 出发日期                   transportDTO.getDepartDateTime()
	        - arriveDateTime 到达日期                   transportDTO.getArriveDateTime()
	        - transferDuration 中转时长                 多航段时
	        	- hour 小时
				- minute 分钟
			- stopList 经停信息                    flightDTO.getStopList() / transportDTO.getDepartDateTime()
				- city 经停城市 结构同上 departCity
				- airport 经停机场 结构同上 departAirport
		  		- duration 经停时长 结构同上 duration
		  		- arriveDateTime 到达时间
			  	- arriveDateTimeEpoch 到达时间戳
			- mainSegment 是否主航段               transportSegment.isMainSegmentInd()           
            - tagList 其他信息
            	- key 键
            	- value 值  
			- virtualFlight 虚拟航班              flightDTO.getVirtualFlightType() / transportSegment    
				- virtualFlightType 虚拟航班类型0：airplane 飞机 1：train 火车 2：car 汽车 3：seaplane 水上飞机
				- transRemarkTitle 中转规则title
				- transRemark 中转规则
				- checkRemarkTitle 值机规则title
				- checkRemark 值机规则
				*概念：虚拟航班=代码共享航班，指两家航空公司有协议，在飞行的时候，挂其中一家航空公司的航班号，实际承运是另一家
			- puIndex PU序号                     PaxSeatDTO::getProductDetailIndex
			- flagList 标签信息                   TransportDTO-RedEye/flightDTO.getMarketingCarrierCode-lcc/flightDTO.getOperatingCarrierCode-shareflight/flightDTO.getStopList-longstay
				- flag 标签
    			- subject 标题
		    	- content 内容
    			- detailList 描述
    				- subject 主题
    				- contentList 内容
    				- action 提示方式（Bullet：弹窗、floating：浮层、None：无）
    		- departDistance 出发城市到主城市的距离信息                需要展示距离时
    		- departMainCity 出发主城市信息 结构同上 departCity
    		- arriveDistance 到达城市到主城市的距离信息
    		- arriveMainCity 到达主城市信息 结构同上 departCity
    	- departAirport 出发机场 结构同上 flightList/departAirport 
    	- arriveAirport 到达机场 结构同上 flightList/departAirport
    	- flagList 标签信息（航程维度） 结构同上 flightList/flagList
    - shoppingId 运价token                                         OfferDTO.getOfferToken() 
   	- seatCount 座位数量                                            PaxSeatDTO
   	- attributes 运价属性                    OfferDTO ItineraryDTO FlightListSearchContext
   		- agencyList 票台       offerDTO.getProductDetailList.getFlightProduct.getFlightSupplier.getAgencyID
   			- id 编号 
   			- code 编码
   		- bookingChannelList 引擎列表      offerDTO.getProductDetailList.getFlightProduct
   			- channel 如WS-CSD            flightProduct.getBookingChannel
   			- engine 如Consolidator       flightProduct.getBookingChannel
   			- validatingCarrier 出票航司   flightProduct.getValidatingCarrier
   		- score 分数                      offerDTO.getItinerary().getPriority()
   		- flagList 标签信息（产品维度） 结构同上 flightList/flagList           OfferDTO ItineraryDTO Context
   		- showNotice 展示信息              OfferDTO Context
   			- cabinClass 舱位
   			- cabinClassLabel 运价包含的所有航段的主舱，以+号分隔，多运价查询时输出？
   			- mainSegmentCabinClass 主舱类型
   		- noticeList 提示信息                      itineraryDTO
   			- flag 标签
    		- subject 标题
		    - content 内容
			    - action 提示方式（Bullet：弹窗、floating：浮层、None：无）
    		- detailList 描述
    			- subject 主题
    			- contentList 内容
    			- action 提示方式（Bullet：弹窗、floating：浮层、None：无）
    	- productDescription 描述信息              OfferDTO Context
    		- packageFlagIcon 打包产品标签图标 推荐航班：recommended 晚出票： lateIssued
    		- packageFlag 打包产品标签
    		- packageCategory 打包产品类型
    		- productName 打包产品类型
    		- productDescription 产品描述
    		- ticketDescription 出票时间描述
   		- mainClassGrade 是否有更高等级的舱等         itineraryDTO
   	- price 价格信息                                OfferDTO
   		- totalPrice 总价                          offerDTO.getViewPrice.getViewTotalPrice
		- averagePrice 均价   isEmpty(context.getPrePrice) ? viewPriceDTO.getViewAveragePrice 
		                                       : viewPriceDTO.getDiffViewAveragePrice(context.getPrePrice())
		- priceDifference 差价                  OfferDTO FlightListSearchContext GetPromoteInfoResponseDTO 
		- discountAveragePrice 折扣均价          viewPriceDTO.getViewAveragePrice() GetPromoteInfoResponseDTO
		- adult
			- salePrice 售价                    viewPriceDTO.getAdultSalePrice
			- tax 税费                          viewPriceDTO.getAdultTax
			- publishPrice 公布运价              viewPriceDTO.getAdultPublishPrice
			- promoFundPrice 后反折扣价          viewPriceDTO.getAdultPromoFundPrice
			- atolPrice Atol价格                viewPriceDTO.getAtolPrice
			- discount 折扣，                     存在公布运价时 就是公布运价-售价，存在promoFundPrice时 就是
			                                     promoFundPrice（for app）							    
			- bookingFee 服务费                   viewPriceDTO.getAdultBookingFeeAmount
			- farePrice 运价价格 有公布运价展示公布运价，没有公布运价展示salePrice+promoFundPrice，再没有展示salePrice
											     viewPriceDTO.getAdultFarePrice
			- totalPrice 单人总价，包含salePrice+tax+atol+bookingFee
			- flightHotelDiscount 机酒折扣金额     viewPriceDTO.getAdultFlightHotelDiscount
			- ticketTotalPrice 单人机票总价 包含farePrice+tax+atol+bookingFee
												 viewPriceDTO.getAdultViewTicketTotalPrice
			- extraFee 加价                       salePrice-publishPrice（for online）
		- child 结构同上 adult
		- infant 结构同上 adult
	- xProduct                                     OfferDTO
		- xProductPreposeList x前置信息             offerDTO
			- policyId 编号
			- description 描述
			- productType 产品类型 commoncoupon servicepackage
			- productName 产品名称
			- token 产品token
			- price 产品价格
			- couponPrice 优惠券价格
			- subProductType 子类型
			- marketingType 营销类型 1：赠送 2：立减 3：定价销售
			- productFullName 产品展示名称 含金额 for online
		- tagList tag信息 key-value
		- transferServiceList Ist中转服务信息     offerDTO
    		- segmentNo 航程号
    		- sequenceNo 航段号
    		- token 产品token
			- salePrice 售价
	- priceKey 运价定位key                        searchResponseDTO, OfferDTO, context
	- needPreLoad 是否需要预加载多舱位、退改签       默认false
	- packageKey 机酒唯一key                      OfferDTO
```

```txt
- resultBasic 基础信息
	- criteriaToken 查询条件
	- currency 币种
	- finish 是否全部输出
	- regionRoute 地区码航线 格式如“CN-JP-TW”
	- lowestPrice 当前查询的最低价 结构同flightProductList/price（从flightProductList中的最低价的产品获取）
	- flightClass I：国际，N：国内
```

```txt
- filterOptionList 过滤可选项
	- segmentNo 行程序号，非合并查询时不输出（什么是合并查询？）
	- airlineList 航司
	- departAirportList 出发机场
	- arriveAirPortList 到达机场
	- stopCityList 经停城市
	- transferCityList 中转城市
	- allianceList 航司联盟（OW:One World/ST:Sky Team/SA:StarAlliance）
    - craftTypeList 机型
    - flagList 标签列表 结构同flightProductList/segmentList/flightList/flagList
    - departCity 出发城市
    	- code 城市三字码
    	- name 城市名
   		- id 城市编号
   		- region 城市所属地区（CN：中国大陆，HK：中国香港，JP:日本等）
   		- timeZone 时区
   		- lowestPrice 最低价
    - arriveCity 到达城市 结构同上出发城市
    - airlineFilterList 根据航司过滤的列表
   		- code 航司三字码
   		- name 航司名称
   		- lcc 是否廉航
   		- alliance 航司联盟
   		- lowestPrice 最低价
    - departAirportFilterList 根据出发机场过滤的列表
       	- code 机场三字码
       	- name 机场名称
       	- city 机场所在的城市 结构同 departCity
    - arriveAirportFliterList 根据到达机场过滤的列表 结构同上根据出发机场过滤
    - stopCityFilterList 根据经停城市过滤的列表 结构同 departCity
    - transferCityFilterList 根据中转城市过滤的列表 结构同上根据经停城市过滤
    - allianceFilterList 根据航司联盟过滤的列表
       	- code 航司联盟二字码？
       	- lowestPrice 最低价
       	- airlineCodeList 航司成员三字码列表
    - aircraftFilterList 根据机型过滤的列表 元素结构同flightProductList/segmentList/flightList/aircraft
```

```txt
- recommend 提示信息 数据来源：flightProductList机票产品信息，recommendFlightProductTypeList 推荐机票产品信息， nearbySuggestResponseDTO 临近城市推荐结果， getHotLowPriceListResponseDTO 低价查询结果， historyDataRetrieveResponseDTO 历史低价结果，pricePredictionResponseDTO 价格预测结果， lowestPrice 当前最低价， context 上下文信息
	- unifyCabinClassList 统一舱等运价产品集合 结构同flightProductList， 将recommendFlightProductTypeList中舱位相同的存储到该list中
	- mixCabinClassList 混合舱等运价产品集合 结构同flightProductList， 将recommendFlightProductTypeList中舱位不同的存储到该list中
	- nearbyCityList 临近城市推荐列表
		- departCity 出发城市信息 结构同flightProductList/segmentList/departCity
		- arriveCity 到达城市信息 结构同flightProductList/segmentList/arriveCity
		- distance 距离
		- nearCity 临近城市 结构同flightProductList/segmentList/departCity
    	- totalPrice 总价
    	- priceDiff 对比当前最低价的价格差
    - lowPrice 低价信息
    	- recommend 推荐信息
    		- departDate 出发日期
    		- arriveDate 到达日期
    		- lowestPrice 推荐最低价
    		- priceDiff 对比当前最低价的价格差
    	- history 历史信息
    		- minHistoryPrice 最低历史价格
    		- maxHistoryPrice 最高历史价格
    		- minHistoryPriceSplit 最低历史分隔价格
    		- maxHistoryPriceSplit 最高历史分隔价格
    		- lowPriceRange 当前低价所在的价格区间 low mid
    		- lowPricePercentage 当前低价所在区间的占位百分比
    		- flagList 提示信息（lowPrice/ Range/ Disclaimer）结构同 flightProductList/segmentList/flagList  
    	- pricePrediction 价格预测
    		- historyPriceList 历史价格信息
    			- searchDate 查询日期
    			- salePrice 售价
    		- futurePriceList 未来价格信息 结构同上
    		- flagList 结构同 flightProductList/segmentList/flagList
    	- directFlightList 直飞航班推荐 直航筛选且筛选无结果时展示
    		- departDate 出发日期
    		- returDate 返程日期（RT航班才有）
    		- salePrice 售价
```

## Request

### API-ListSearchFirst（Streaming）
列表查询请求
一共请求两次，第一次加锁写缓存（主线程返回第一批数据，子线程等待所有数据到达写缓存），第二次解锁读缓存
```json
{
  "Head": {
    "locale": "en-GB",
    "source": "WAP",
    "version": "500",
    "currency": "EUR",
    "uID": "E4827758298",
    "vID": "1626262136061.32qx10",
    "iP": "ed5d555eb0db65948684675d0c16b64076c2cad7950fd42614e2d72d2f60c170",
    "clientID": "",
    "deviceID": "",
    "ticket": "ED3AF6A1A6F764CE0B764754CB8D34A51586154D564E901FB2FDD7803425E303",
    "token": "",
    "allianceInfo": {
      "allianceID": 2175,
      "sID": 957211,
      "ouID": "mIp8KHOGEey28AJCrBEAEQ",
      "uuid": "",
      "useDistributionType": 1
    },
    "transactionID": "20220207151136221",
    "guest": false,
    "group": "Trip",
    "extendFields": {
      "BatchedId": "78b6870c-5a4c-460e-a123-3cd070a20ff5",
      "Member": "1"
    },
    "channel": "UKSite",
    "pvId": "2603",
    "sessionId": "177",
    "abTesting": "",
    "clientAppId": null,
    "isQuickBooking": null,
    "clientTime": null,
    "productLine": null
  },
  "searchCriteria": {
    "cabinClass": "YSGroup",
    "passengerCount": {
      "adult": 1,
      "child": 0,
      "infant": 0
    },
    "searchSegmentList": [
      {
        "departDate": "2022-02-28",
        "departCity": "LON",
        "arriveCity": "NYC",
        "dCityVirtualRegionCode": null,
        "aCityVirtualRegionCode": null
      },
      {
        "departDate": "2022-03-03",
        "departCity": "NYC",
        "arriveCity": "LON",
        "dCityVirtualRegionCode": null,
        "aCityVirtualRegionCode": null
      }
    ]
  },
  "criteriaToken": "",
  "shoppingId": "",
  "priceKey": null,
  "segmentNo": 1,
  "filter": {
    "filterSegmentList": [
      {
        "segmentNo": null,
        "airlineList": [
          ""
        ],
        "departAirportList": null,
        "arriveAirportList": null,
        "departTimeSpanList": null,
        "arriveTimeSpanList": null,
        "allianceList": null,
        "transferCityList": null,
        "craftTypeList": null
      }
    ],
    "flagList": null,
    "promotionId": null
  },
  "sort": null,
  "pagination": null,
  "fullData": false,
  "clientTagList": null,
  "clientParameterTagList": null,
  "tagList": null,
  "searchMode": null
}
```

### AGG-ListSearchFirst（Streaming）

```json
{
  "RequestHeader": {
    "MessageHeader": {
      "Channel": "EnglishSite",
      "SubChannelID": 8,
      "TransactionID": "20220207155925366",
      "RequestID": "20220207155925366",
      "ClientIP": "45.251.105.183",
      "DevicePlatform": "WAP",
      "SessionID": "178"
    }
  },
  "SearchCriteria": {
    "JourneyRequest": [
      {
        "DepartDate": "2022-02-28",
        "DepartLocation": {
          "Type": 1,
          "Location": "LON"
        },
        "ArriveLocation": {
          "Type": 1,
          "Location": "NYC"
        }
      },
      {
        "DepartDate": "2022-03-03",
        "DepartLocation": {
          "Type": 1,
          "Location": "NYC"
        },
        "ArriveLocation": {
          "Type": 1,
          "Location": "LON"
        }
      }
    ],
    "PaxRequest": [
      {
        "AgeType": 1,
        "Count": 1
      }
    ],
    "CabinCode": [
      "Y",
      "W"
    ],
    "SaleCurrency": "EUR",
    "Language": "EN_GB",
    "UserParameter": {
      "UserID": "E4827758298"
    },
    "IntlOrDomesticRequest": 1
  }
}
```

### API-ListSearchFirst（Soa）

```txt

{
  "Head": {
    "locale": "en-us",
    "source": "ONLINE",
    "version": "703.000",
    "currency": "HKD",
    "uID": null,
    "vID": null,
    "iP": "e9348c510160f14fd41b5dec378c8f24cd6d6abdceb7fbd3cce70e32d350b5e1",
    "clientID": null,
    "deviceID": null,
    "ticket": null,
    "token": null,
    "allianceInfo": null,
    "transactionID": "1644218775449",
    "guest": null,
    "group": "trip",
    "extendFields": null,
    "channel": null,
    "pvId": null,
    "sessionId": null,
    "abTesting": null,
    "clientAppId": null,
    "isQuickBooking": null,
    "clientTime": null,
    "productLine": null
  },
  "searchCriteria": {
    "cabinClass": "YSGroup",
    "passengerCount": {
      "adult": 1,
      "child": 0,
      "infant": 0
    },
    "searchSegmentList": [
      {
        "departDate": "2022-04-13",
        "departCity": "HKG",
        "arriveCity": "MNL",
        "dCityVirtualRegionCode": null,
        "aCityVirtualRegionCode": null
      }
    ]
  },
  "criteriaToken": null,
  "shoppingId": null,
  "priceKey": null,
  "segmentNo": null,
  "filter": null,
  "sort": {
    "type": "Score",
    "asc": true,
    "topChosenPrice": null,
    "topAirline": null,
    "topList": null
  },
  "pagination": null,
  "fullData": true<1>,
  "clientTagList": null,
  "clientParameterTagList": null,
  "tagList": null,
  "searchMode": null
}
# <1> 只请求一次
```

### API-ListSearchNext（Soa）

```json
{
  "Head": {
    "locale": "en-GB",
    "source": "WAP",
    "version": "500",
    "currency": "EUR",
    "uID": "E4827758298",
    "vID": "1626262136061.32qx10",
    "iP": "ed5d555eb0db65948684675d0c16b64076c2cad7950fd42614e2d72d2f60c170",
    "clientID": "",
    "deviceID": "",
    "ticket": "ED3AF6A1A6F764CE0B764754CB8D34A51586154D564E901FB2FDD7803425E303",
    "token": "",
    "allianceInfo": {
      "allianceID": 2175,
      "sID": 957211,
      "ouID": "mIp8KHOGEey28AJCrBEAEQ",
      "uuid": "",
      "useDistributionType": 1
    },
    "transactionID": "20220207191731507",
    "guest": false,
    "group": "Trip",
    "extendFields": {
      "BatchedId": "f37d8639-08a1-42c7-953c-f19165fcb614",
      "Member": "1"
    },
    "channel": "UKSite",
    "pvId": "2633",
    "sessionId": "182",
    "abTesting": "",
    "clientAppId": null,
    "isQuickBooking": null,
    "clientTime": null,
    "productLine": null
  },
  "searchCriteria": {
    "cabinClass": "YSGroup",
    "passengerCount": {
      "adult": 1,
      "child": 0,
      "infant": 0
    },
    "searchSegmentList": [
      {
        "departDate": "2022-02-28",
        "departCity": "LON",
        "arriveCity": "NYC",
        "dCityVirtualRegionCode": null,
        "aCityVirtualRegionCode": null
      },
      {
        "departDate": "2022-03-07",
        "departCity": "NYC",
        "arriveCity": "LON",
        "dCityVirtualRegionCode": null,
        "aCityVirtualRegionCode": null
      }
    ]
  },
  "criteriaToken": "tripType:RT|cabinClass:YSGroup|adult:1|child:0|infant:0|subChannel:8|channel:EnglishSite|currency:EUR|list:true|idc:SHAXY|date_1:2022-02-28|aCity_1:NYC|dCity_1:LON|date_2:2022-03-07|aCity_2:LON|dCity_2:NYC|vsType:3",
  "shoppingId": "8000000128K3m09y05z2qn002WW1mvQOY029m0uWjCXF0j00GcMcGdW80000000Jm000G0000000100042I56G0024AIee00ZDy800G1P2qq6J",
  "priceKey": null,
  "segmentNo": 2,
  "filter": {
    "filterSegmentList": [
      {
        "segmentNo": null,
        "airlineList": [
          ""
        ],
        "departAirportList": null,
        "arriveAirportList": null,
        "departTimeSpanList": null,
        "arriveTimeSpanList": null,
        "allianceList": null,
        "transferCityList": null,
        "craftTypeList": null
      }
    ],
    "flagList": null,
    "promotionId": null
  },
  "sort": null,
  "pagination": null,
  "fullData": false,
  "clientTagList": null,
  "clientParameterTagList": null,
  "tagList": null,
  "searchMode": null
}
```

### AGG-ListSearchNext（Soa）

```json
{
  "RequestHeader": {
    "MessageHeader": {
      "Channel": "EnglishSite",
      "SubChannelID": 8,
      "TransactionID": "20220207191731507",
      "RequestID": "20220207191731507",
      "ClientIP": "45.251.105.183",
      "DevicePlatform": "WAP",
      "SessionID": "182"
    }
  },
  "SearchCriteria": {
    "JourneyRequest": [
      {
        "DepartDate": "2022-02-28",
        "DepartLocation": {
          "Type": 1,
          "Location": "LON"
        },
        "ArriveLocation": {
          "Type": 1,
          "Location": "NYC"
        }
      },
      {
        "DepartDate": "2022-03-07",
        "DepartLocation": {
          "Type": 1,
          "Location": "NYC"
        },
        "ArriveLocation": {
          "Type": 1,
          "Location": "LON"
        }
      }
    ],
    "PaxRequest": [
      {
        "AgeType": 1,
        "Count": 1
      }
    ],
    "CabinCode": [
      "Y",
      "W"
    ],
    "SaleCurrency": "EUR",
    "Language": "EN_GB",
    "UserParameter": {
      "UserID": "E4827758298"
    },
    "IntlOrDomesticRequest": 1
  },
  "RouteSearch": {
    "SearchMode": 1,
    "MaxJourneyNo": 1,
    "OfferToken": "8000000128K3m09y05z2qn002WW1mvQOY029m0uWjCXF0j00GcMcGdW80000000Jm000G0000000100042I56G0024AIee00ZDy800G1P2qq6J"
  }
}
```

### API-GroupSearch

```json
{
  "Head": {
    "locale": "en-GB",
    "source": "WAP",
    "version": "500",
    "currency": "EUR",
    "uID": "E4827758298",
    "vID": "1626262136061.32qx10",
    "iP": "ed5d555eb0db65948684675d0c16b64076c2cad7950fd42614e2d72d2f60c170",
    "clientID": "",
    "deviceID": "",
    "ticket": "ED3AF6A1A6F764CE0B764754CB8D34A51586154D564E901FB2FDD7803425E303",
    "token": "",
    "allianceInfo": {
      "allianceID": 2175,
      "sID": 957211,
      "ouID": "mIp8KHOGEey28AJCrBEAEQ",
      "uuid": "",
      "useDistributionType": 1
    },
    "transactionID": "20220207154649811",
    "guest": false,
    "group": "Trip",
    "extendFields": {
      "BatchedId": "faa92297-5d3e-4ab1-8542-8fa97cee8a15"
    },
    "channel": "UKSite",
    "pvId": "2607",
    "sessionId": "178",
    "abTesting": "",
    "clientAppId": null,
    "isQuickBooking": null,
    "clientTime": null,
    "productLine": null
  },
  "criteriaToken": "tripType:OW|cabinClass:YSGroup|adult:1|child:0|infant:0|subChannel:8|channel:EnglishSite|currency:EUR|list:true|idc:SHAXY|date_1:2022-02-28|aCity_1:NYC|dCity_1:LON|vsType:3",
  "shoppingId": "8000000128I2mmLy0W2mLH0OCE0G50AeY8000000001B00G00000001000TsLL4m006gAIee00TiS8001vomLGs2",
  "priceKey": null,
  "filter": null,
  "moreCabinClass": false,
  "searchMode": null,
  "searchCriteria": null,
  "tagList": null
}
```

### AGG-GroupSearch

```json
{
  "RequestHeader": {
    "MessageHeader": {
      "Channel": "EnglishSite",
      "SubChannelID": 8,
      "TransactionID": "20220207154649811",
      "RequestID": "20220207154649811",
      "ClientIP": "45.251.105.183",
      "DevicePlatform": "WAP",
      "SessionID": "178"
    }
  },
  "SearchCriteria": {
    "JourneyRequest": [
      {
        "DepartDate": "2022-02-28",
        "DepartLocation": {
          "Type": 1,
          "Location": "LON"
        },
        "ArriveLocation": {
          "Type": 1,
          "Location": "NYC"
        }
      }
    ],
    "PaxRequest": [
      {
        "AgeType": 1,
        "Count": 1
      }
    ],
    "CabinCode": [
      "Y",
      "W"
    ],
    "SaleCurrency": "EUR",
    "Language": "EN_GB",
    "UserParameter": {
      "UserID": "E4827758298"
    },
    "IntlOrDomesticRequest": 1
  },
  "RouteSearch": {
    "SearchMode": 1,
    "MaxJourneyNo": 1,
    "OfferToken": "8000000128I2mmLy0W2mLH0OCE0G50AeY8000000001B00G00000001000TsLL4m006gAIee00TiS8001vomLGs2"
  }
}
```

### API-DetailSearch

```json
{
  "Head": {
    "locale": "en-GB",
    "source": "WAP",
    "version": "500",
    "currency": "EUR",
    "uID": "E4827758298",
    "vID": "1626262136061.32qx10",
    "iP": "ed5d555eb0db65948684675d0c16b64076c2cad7950fd42614e2d72d2f60c170",
    "clientID": "",
    "deviceID": "",
    "ticket": "ED3AF6A1A6F764CE0B764754CB8D34A51586154D564E901FB2FDD7803425E303",
    "token": "",
    "allianceInfo": {
      "allianceID": 2175,
      "sID": 957211,
      "ouID": "mIp8KHOGEey28AJCrBEAEQ",
      "uuid": "",
      "useDistributionType": 1
    },
    "transactionID": "20220207160923204",
    "guest": false,
    "group": "Trip",
    "extendFields": {
      "BatchedId": "aea7d874-054f-4001-8c72-9514b6e17b32"
    },
    "channel": "UKSite",
    "pvId": "2613",
    "sessionId": "178",
    "abTesting": "",
    "clientAppId": null,
    "isQuickBooking": null,
    "clientTime": null,
    "productLine": null
  },
  "criteriaToken": "tripType:OW|cabinClass:YSGroup|adult:1|child:0|infant:0|subChannel:8|channel:EnglishSite|currency:EUR|list:true|idc:SHAXY|date_1:2022-02-28|aCity_1:NYC|dCity_1:LON|vsType:3",
  "shoppingId": "8000000128I0Y33S0bIWLH002-01u8AeY800000000120000000000W009EI510G000oAIee00Rl8801DYBGLK31",
  "filter": null,
  "searchMode": null,
  "searchCriteria": null,
  "priceKey": null,
  "tagList": null,
  "disableCache": false
}
```

### AGG-DetailSearch

```txt
{
  "RequestHeader": {
    "MessageHeader": {
      "Channel": "EnglishSite",
      "SubChannelID": 8,
      "TransactionID": "20220207160923204",
      "RequestID": "20220207160923204",
      "ClientIP": "45.251.105.183",
      "DevicePlatform": "WAP",
      "SessionID": "178"
    }
  },
  "SearchCriteria": {
    "JourneyRequest": [
      {
        "DepartDate": "2022-02-28",
        "DepartLocation": {
          "Type": 1,
          "Location": "LON"
        },
        "ArriveLocation": {
          "Type": 1,
          "Location": "NYC"
        }
      }
    ],
    "PaxRequest": [
      {
        "AgeType": 1,
        "Count": 1
      }
    ],
    "CabinCode": [
      "Y",
      "W"
    ],
    "SaleCurrency": "EUR",
    "Language": "EN_GB",
    "UserParameter": {
      "UserID": "E4827758298"
    },
    "IntlOrDomesticRequest": 1
  },
  "RouteSearch": {
    "SearchMode": 2<1>,
    "OfferToken": "8000000128I0Y33S0bIWLH002-01u8AeY800000000120000000000W009EI510G000oAIee00Rl8801DYBGLK31"
  }
}
# <1>SearchMode == 2 和列表查询，中间页查询 SearchMode == 1 不同 =》严格反查
```

### API-XlistSearch

```json
{
  "Head": {
    "locale": "en-GB",
    "source": "WAP",
    "version": "500",
    "currency": "EUR",
    "uID": "E4827758298",
    "vID": "1626262136061.32qx10",
    "iP": "ed5d555eb0db65948684675d0c16b64076c2cad7950fd42614e2d72d2f60c170",
    "clientID": "",
    "deviceID": "",
    "ticket": "ED3AF6A1A6F764CE0B764754CB8D34A51586154D564E901FB2FDD7803425E303",
    "token": "",
    "allianceInfo": {
      "allianceID": 2175,
      "sID": 957211,
      "ouID": "mIp8KHOGEey28AJCrBEAEQ",
      "uuid": "",
      "useDistributionType": 1
    },
    "transactionID": "20220207160923204",
    "guest": false,
    "group": "Trip",
    "extendFields": {
      "BatchedId": "aea7d874-054f-4001-8c72-9514b6e17b32"
    },
    "channel": "UKSite",
    "pvId": "2614",
    "sessionId": "178",
    "abTesting": "",
    "clientAppId": null,
    "isQuickBooking": null,
    "clientTime": null,
    "productLine": null
  },
  "productTypeList": [
    "Baggage",
    "Insurance",
    "VirtualService",
    "ServicePackage",
    "BookSeat"
  ],
  "queryCondition": {
    "insuranceSpecific": {
      "underWriteStartTime": "2022-02-28",
      "underWriteEndTime": "2022-03-02"
    },
    "servicePackageSpecific": null,
    "bookSeatSpecific": null
  },
  "criteriaToken": "tripType:OW|cabinClass:YSGroup|adult:1|child:0|infant:0|subChannel:8|channel:EnglishSite|currency:EUR|list:true|idc:SHAXY|date_1:2022-02-28|aCity_1:NYC|dCity_1:LON|vsType:3",
  "shoppingId": "8000000128I0Y33S0bIWLH002-01u8AeY800000000120000000000W009EI510G000oAIee00Rl8801DYBGLK31",
  "passengerList": null,
  "priceKey": null
}
```

### AGG-XlistSearch

```json
{
  "RequestHeader": {
    "MessageHeader": {
      "Channel": "EnglishSite",
      "SubChannelID": 8,
      "TransactionID": "20220207160923204",
      "RequestID": "20220207160923204",
      "ClientIP": "45.251.105.183",
      "DevicePlatform": "WAP",
      "SessionID": "178"
    }
  },
  "SearchCriteria": {
    "JourneyRequest": [
      {
        "DepartDate": "2022-02-28",
        "DepartLocation": {
          "Type": 1,
          "Location": "LON"
        },
        "ArriveLocation": {
          "Type": 1,
          "Location": "NYC"
        }
      }
    ],
    "PaxRequest": [
      {
        "AgeType": 1,
        "Count": 1
      }
    ],
    "CabinCode": [
      "Y",
      "W"
    ],
    "SaleCurrency": "EUR",
    "Language": "EN_GB",
    "UserParameter": {
      "UserID": "E4827758298"
    },
    "IntlOrDomesticRequest": 1
  },
  "RouteSearch": {
    "SearchMode": 2,
    "OfferToken": "8000000128I0Y33S0bIWLH002-01u8AeY800000000120000000000W009EI510G000oAIee00Rl8801DYBGLK31"
  }
}
```

## Response

主要看search，分析30639 请求的agg 大交通search 接口的契约

存在疑问的字段：

mainSegmentInd、PUSequence、FCSequence

作用是什么？字段应该是底层ATPCO engine 返回的，看engine中的对应部分

