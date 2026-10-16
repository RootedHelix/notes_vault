
`(.venv) PS C:\Users\samo1\Desktop\programming\projects\currency_analysis\PriceSynergies> python scripts/sweep_env.py --engine ensemble --runs 3`  
`[sweep] TOTAL START @ 2026-01-07 14:07:31`  
`[sweep] Run 1/3 -> {'ML_N_LAGS': '12', 'ML_ROLL_WINDOWS': '6,12,24,72,168', 'ML_VOL_WINDOWS': '24,72,168', 'ML_HORIZON': '6', 'ML_MATCH_HORIZON': '0', 'TRAIN_SELECT_METRIC': 'pnl_sharpe', 'TRAIN_PNL_BAND': '0.0', 'TRAIN_FEATURE_SCOPE': 'all', 'TRAIN_FEATURE_EXTRA_PREFIXES': 'regime_,spread_', 'TRAIN_TOP_FEATURES': '300', 'TRAIN_MIN_ABS_CORR': '0.0', 'TRAIN_TS_SPLITS': '5', 'TRAIN_TS_GAP': '0', 'TRAIN_SEARCH_ITER': '25', 'TRAIN_N_JOBS': '-1', 'ML_TA_RSI': '0', 'ML_RSI_WINDOWS': '7,14,28', 'ML_TA_MACD': '0', 'ML_MACD_FAST': '12', 'ML_MACD_SLOW': '21', 'ML_MACD_SIGNAL': '9', 'ML_TA_TREND': '1', 'ML_TREND_WINDOWS': '10,20', 'ML_TA_SR': '0', 'ML_SR_WINDOWS': '20,50', 'ML_USE_OHLCV': '1', 'ML_TA_SMA': '1', 'ML_TA_EMA': '0', 'ML_TA_BB': '0', 'ML_TA_ATR': '1', 'ML_TA_STOCH': '0', 'ML_TA_CCI': '0', 'ML_TA_OBV': '1', 'ML_TA_CMF': '0', 'ML_TA_ROC': '1', 'ML_TA_RANGE_POS': '1', 'ML_SMA_LEN': '30', 'ML_EMA_LEN': '20', 'ML_BB_LEN': '10', 'ML_BB_STD': '2.5', 'ML_ATR_LEN': '14', 'ML_STOCH_K_LEN': '20', 'ML_STOCH_D_LEN': '3', 'ML_CCI_LEN': '20', 'ML_ROC_LEN': '6', 'ML_RANGE_POS_LEN': '40', 'ML_CMF_LEN': '20', 'SIGNAL_BAND': '0.01', 'EXIT_BAND': '0.0005', 'EXIT_HOLD_BARS': '2', 'EXIT_STOP_PCT': '0.0', 'EXIT_TAKE_PCT': '0.0', 'BACKTEST_LAGS': '36', 'BACKTEST_TRAIN_FRAC': '0.5', 'SIGNAL_STRATEGY': 'strat_longonly_ret', 'ML_CONF_SCALE': '0.0', 'ML_CONF_FULL': '0.5', 'ENSEMBLE_CONF_SCALE': '0.0', 'ENSEMBLE_CONF_FULL': '0.5', 'SIZE_RULE_ENABLE': '1', 'SIZE_RULE_FLOOR': '0.65', 'SIZE_RULE_MID': '0.8', 'SIZE_RULE_TOP': '0.9', 'SIZE_RULE_MIN_MULT': '0.0', 'SIZE_RULE_MID_MULT': '1.0', 'SIZE_RULE_MAX_MULT': '3.0', 'ENSEMBLE_WEIGHT_ML': '0.2', 'ENSEMBLE_WEIGHT_VAR': '0.8', 'ENSEMBLE_BAND': '0.0'}`  
`[sweep] prepare_ml START ['python', 'scripts/prepare_ml.py'] @ 2026-01-07 14:07:31`  
`Final dataset shape after target drop/fill: (3652, 1060)`  
`Saved train -> output\ml_train.parquet (2336 rows), val -> output\ml_val.parquet (585 rows), backtest -> output\ml_backtest.parquet (731 rows)`  
`Done.`  
`[sweep] prepare_ml END rc=0 in 2.4s @ 2026-01-07 14:07:34`  
`[sweep] train_model START ['python', 'scripts/train_model.py'] @ 2026-01-07 14:07:34`  
`target_Gold_h6: random_forest CV MAE 0.756651 (std 0.18472323144014408) params={'rf__n_estimators': 600, 'rf__min_samples_split': 4, 'rf__min_samples_leaf': 2, 'rf__max_samples': 0.6, 'rf__max_features': 0.7, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`[sweep] train_model still running... 60.0s elapsed`  
`target_Gold_h6: hist_gradient_boosting CV MAE 0.775477 (std 0.16265960443721172) params={'min_samples_leaf': 50, 'max_leaf_nodes': 127, 'max_depth': 9, 'max_bins': 255, 'learning_rate': 0.03, 'l2_regularization': 0.01}`  
`[sweep] train_model still running... 120.0s elapsed`  
`[sweep] train_model still running... 180.0s elapsed`  
`target_Gold_h6: xgboost CV MAE 0.757428 (std 0.1619455956645453) params={'subsample': 0.6, 'reg_lambda': 0.1, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_Gold_h6: mlp_regressor CV MAE 1.066310 (std 0.20792983380116267) params={'mlp__learning_rate_init': 0.001, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 1e-05, 'mlp__activation': 'tanh'}`  
`target_Gold_h6: selected random_forest with CV MAE 0.756651`  
`Saved model -> models\model_target_Gold_h6.pkl`  
`Train MAE: 0.385034`  
`Val MAE: 0.807399`  
`[sweep] train_model still running... 240.0s elapsed`  
`target_Silver_h6: random_forest CV MAE 1.371225 (std 0.49697246267351536) params={'rf__n_estimators': 600, 'rf__min_samples_split': 4, 'rf__min_samples_leaf': 2, 'rf__max_samples': 0.6, 'rf__max_features': 0.7, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_Silver_h6: hist_gradient_boosting CV MAE 1.417540 (std 0.4599833554051363) params={'min_samples_leaf': 50, 'max_leaf_nodes': 127, 'max_depth': 9, 'max_bins': 255, 'learning_rate': 0.03, 'l2_regularization': 0.01}`  
`[sweep] train_model still running... 300.0s elapsed`  
`target_Silver_h6: xgboost CV MAE 1.464447 (std 0.5571903825008212) params={'subsample': 0.6, 'reg_lambda': 0.1, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`[sweep] train_model still running... 360.0s elapsed`  
`target_Silver_h6: mlp_regressor CV MAE 2.048673 (std 0.5877407773221157) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.001, 'mlp__activation': 'tanh'}`  
`target_Silver_h6: selected random_forest with CV MAE 1.371225`  
`Saved model -> models\model_target_Silver_h6.pkl`  
`Train MAE: 0.712407`  
`Val MAE: 1.689896`  
`[sweep] train_model still running... 420.0s elapsed`  
`target_Bitcoin_h6: random_forest CV MAE 6.853063 (std 2.7094025641652277) params={'rf__n_estimators': 400, 'rf__min_samples_split': 8, 'rf__min_samples_leaf': 12, 'rf__max_samples': 0.6, 'rf__max_features': 0.5, 'rf__max_depth': 4, 'rf__bootstrap': True}`  
`target_Bitcoin_h6: hist_gradient_boosting CV MAE 6.998809 (std 2.637695357731947) params={'min_samples_leaf': 100, 'max_leaf_nodes': 31, 'max_depth': 3, 'max_bins': 63, 'learning_rate': 0.03, 'l2_regularization': 0.01}`  
`[sweep] train_model still running... 480.0s elapsed`  
`[sweep] train_model still running... 540.0s elapsed`  
`target_Bitcoin_h6: xgboost CV MAE 7.458722 (std 3.8293683949180215) params={'subsample': 0.6, 'reg_lambda': 0.01, 'min_child_weight': 5, 'max_depth': 9, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_Bitcoin_h6: mlp_regressor CV MAE 11.430555 (std 3.1084120944253417) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (64, 32), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.0001, 'mlp__activation': 'tanh'}`  
`target_Bitcoin_h6: selected hist_gradient_boosting with CV MAE 6.998809`  
`Saved model -> models\model_target_Bitcoin_h6.pkl`  
`Train MAE: 4.463908`  
`Val MAE: 2.821318`  
`[sweep] train_model still running... 600.0s elapsed`  
`[sweep] train_model still running... 660.0s elapsed`  
`target_ETH_h6: random_forest CV MAE 5.684842 (std 1.3937358078112163) params={'rf__n_estimators': 600, 'rf__min_samples_split': 8, 'rf__min_samples_leaf': 8, 'rf__max_samples': 0.8, 'rf__max_features': 0.7, 'rf__max_depth': 8, 'rf__bootstrap': True}`  
`[sweep] train_model still running... 720.0s elapsed`  
`target_ETH_h6: hist_gradient_boosting CV MAE 5.917190 (std 1.4914759533937219) params={'min_samples_leaf': 50, 'max_leaf_nodes': 127, 'max_depth': 3, 'max_bins': 127, 'learning_rate': 0.05, 'l2_regularization': 0.01}`  
`[sweep] train_model still running... 780.0s elapsed`  
`target_ETH_h6: xgboost CV MAE 5.757773 (std 1.3175798170032986) params={'subsample': 0.8, 'reg_lambda': 1.0, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.03, 'colsample_bytree': 0.8}`  
`[sweep] train_model still running... 840.0s elapsed`  
`target_ETH_h6: mlp_regressor CV MAE 7.923277 (std 2.3604774081905866) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (128,), 'mlp__batch_size': 'auto', 'mlp__alpha': 1e-05, 'mlp__activation': 'tanh'}`  
`target_ETH_h6: selected random_forest with CV MAE 5.684842`  
`Saved model -> models\model_target_ETH_h6.pkl`  
`Train MAE: 2.309008`  
`Val MAE: 3.608961`  
`[sweep] train_model still running... 900.0s elapsed`  
`[sweep] train_model still running... 960.0s elapsed`  
`target_XRP_h6: random_forest CV MAE 8.107165 (std 2.2653385398386487) params={'rf__n_estimators': 400, 'rf__min_samples_split': 8, 'rf__min_samples_leaf': 8, 'rf__max_samples': 0.5, 'rf__max_features': 0.7, 'rf__max_depth': 6, 'rf__bootstrap': True}`  
`target_XRP_h6: hist_gradient_boosting CV MAE 8.731635 (std 1.9636964392070484) params={'min_samples_leaf': 50, 'max_leaf_nodes': 127, 'max_depth': 9, 'max_bins': 255, 'learning_rate': 0.03, 'l2_regularization': 0.01}`  
`[sweep] train_model still running... 1020.0s elapsed`  
`target_XRP_h6: xgboost CV MAE 9.199902 (std 1.82303536982099) params={'subsample': 0.8, 'reg_lambda': 1.0, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.03, 'colsample_bytree': 0.8}`  
`[sweep] train_model still running... 1080.0s elapsed`  
`target_XRP_h6: mlp_regressor CV MAE 11.751307 (std 2.6038046717059036) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (128,), 'mlp__batch_size': 'auto', 'mlp__alpha': 1e-05, 'mlp__activation': 'tanh'}`  
`target_XRP_h6: selected random_forest with CV MAE 8.107165`  
`Saved model -> models\model_target_XRP_h6.pkl`  
`Train MAE: 4.766233`  
`Val MAE: 5.190545`  
`target_SP500_h6: random_forest CV MAE 0.259220 (std 0.11559280652202301) params={'rf__n_estimators': 600, 'rf__min_samples_split': 6, 'rf__min_samples_leaf': 4, 'rf__max_samples': 0.8, 'rf__max_features': 0.5, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_SP500_h6: hist_gradient_boosting CV MAE 0.275246 (std 0.10221759047630904) params={'min_samples_leaf': 10, 'max_leaf_nodes': 15, 'max_depth': 7, 'max_bins': 127, 'learning_rate': 0.05, 'l2_regularization': 0.05}`  
`[sweep] train_model still running... 1140.0s elapsed`  
`[sweep] train_model still running... 1200.0s elapsed`  
`target_SP500_h6: xgboost CV MAE 0.191281 (std 0.08425472938845574) params={'subsample': 0.6, 'reg_lambda': 0.1, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_SP500_h6: mlp_regressor CV MAE 0.769770 (std 0.19159775763241824) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.001, 'mlp__activation': 'tanh'}`  
`target_SP500_h6: selected xgboost with CV MAE 0.191281`  
`Saved model -> models\model_target_SP500_h6.pkl`  
`Train MAE: 0.011960`  
`Val MAE: 0.113131`  
`[sweep] train_model still running... 1260.0s elapsed`  
`target_Nasdaq100_h6: random_forest CV MAE 0.501216 (std 0.09091185803338467) params={'rf__n_estimators': 600, 'rf__min_samples_split': 4, 'rf__min_samples_leaf': 2, 'rf__max_samples': 0.6, 'rf__max_features': 0.7, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_Nasdaq100_h6: hist_gradient_boosting CV MAE 0.517883 (std 0.10238278084066009) params={'min_samples_leaf': 10, 'max_leaf_nodes': 15, 'max_depth': 7, 'max_bins': 127, 'learning_rate': 0.05, 'l2_regularization': 0.05}`  
`[sweep] train_model still running... 1320.0s elapsed`  
`[sweep] train_model still running... 1380.0s elapsed`  
`target_Nasdaq100_h6: xgboost CV MAE 0.475265 (std 0.07615197166556731) params={'subsample': 1.0, 'reg_lambda': 1.0, 'min_child_weight': 1, 'max_depth': 3, 'learning_rate': 0.1, 'colsample_bytree': 0.8}`  
`target_Nasdaq100_h6: mlp_regressor CV MAE 1.316384 (std 0.2744061944232563) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 1e-05, 'mlp__activation': 'tanh'}`  
`target_Nasdaq100_h6: selected hist_gradient_boosting with CV MAE 0.517883`  
`Saved model -> models\model_target_Nasdaq100_h6.pkl`  
`Train MAE: 0.250616`  
`Val MAE: 0.434775`  
`[sweep] train_model still running... 1440.0s elapsed`  
`target_USSmallCap_h6: random_forest CV MAE 1.000906 (std 0.23758460761640762) params={'rf__n_estimators': 600, 'rf__min_samples_split': 6, 'rf__min_samples_leaf': 4, 'rf__max_samples': 0.8, 'rf__max_features': 0.5, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_USSmallCap_h6: hist_gradient_boosting CV MAE 0.992142 (std 0.23670121331326352) params={'min_samples_leaf': 20, 'max_leaf_nodes': 15, 'max_depth': 7, 'max_bins': 255, 'learning_rate': 0.05, 'l2_regularization': 0.001}`  
`[sweep] train_model still running... 1500.0s elapsed`  
`[sweep] train_model still running... 1560.0s elapsed`  
`target_USSmallCap_h6: xgboost CV MAE 0.941494 (std 0.23424194362621917) params={'subsample': 0.6, 'reg_lambda': 0.1, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_USSmallCap_h6: mlp_regressor CV MAE 1.313657 (std 0.26442755723262745) params={'mlp__learning_rate_init': 0.001, 'mlp__hidden_layer_sizes': (128,), 'mlp__batch_size': 'auto', 'mlp__alpha': 1e-05, 'mlp__activation': 'relu'}`  
`target_USSmallCap_h6: selected xgboost with CV MAE 0.941494`  
`Saved model -> models\model_target_USSmallCap_h6.pkl`  
`Train MAE: 0.064693`  
`Val MAE: 0.854293`  
`target_TotalUS_h6: random_forest CV MAE 0.217056 (std 0.13631813985772379) params={'rf__n_estimators': 600, 'rf__min_samples_split': 6, 'rf__min_samples_leaf': 4, 'rf__max_samples': 0.8, 'rf__max_features': 0.5, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`[sweep] train_model still running... 1620.0s elapsed`  
`target_TotalUS_h6: hist_gradient_boosting CV MAE 0.265641 (std 0.11154501531486047) params={'min_samples_leaf': 10, 'max_leaf_nodes': 15, 'max_depth': 7, 'max_bins': 127, 'learning_rate': 0.05, 'l2_regularization': 0.05}`  
`[sweep] train_model still running... 1680.0s elapsed`  
`target_TotalUS_h6: xgboost CV MAE 0.172455 (std 0.09895733412967105) params={'subsample': 0.8, 'reg_lambda': 1.0, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.03, 'colsample_bytree': 0.8}`  
`target_TotalUS_h6: mlp_regressor CV MAE 0.731216 (std 0.2222538337391061) params={'mlp__learning_rate_init': 0.001, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.001, 'mlp__activation': 'relu'}`  
`target_TotalUS_h6: selected random_forest with CV MAE 0.217056`  
`Saved model -> models\model_target_TotalUS_h6.pkl`  
`Train MAE: 0.071620`  
`Val MAE: 0.098066`  
`[sweep] train_model still running... 1740.0s elapsed`  
`target_IntlDev_h6: random_forest CV MAE 0.713799 (std 0.11395943711542746) params={'rf__n_estimators': 600, 'rf__min_samples_split': 6, 'rf__min_samples_leaf': 8, 'rf__max_samples': 0.5, 'rf__max_features': 0.3, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_IntlDev_h6: hist_gradient_boosting CV MAE 0.735700 (std 0.1201712794201944) params={'min_samples_leaf': 50, 'max_leaf_nodes': 127, 'max_depth': 9, 'max_bins': 255, 'learning_rate': 0.03, 'l2_regularization': 0.01}`  
`[sweep] train_model still running... 1800.0s elapsed`  
`[sweep] train_model still running... 1860.0s elapsed`  
`target_IntlDev_h6: xgboost CV MAE 0.743144 (std 0.08531459417198407) params={'subsample': 0.6, 'reg_lambda': 0.1, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_IntlDev_h6: mlp_regressor CV MAE 1.013425 (std 0.1467896214610114) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.001, 'mlp__activation': 'relu'}`  
`target_IntlDev_h6: selected hist_gradient_boosting with CV MAE 0.735700`  
`Saved model -> models\model_target_IntlDev_h6.pkl`  
`Train MAE: 0.446463`  
`Val MAE: 0.683828`  
`target_Emerging_h6: random_forest CV MAE 1.097568 (std 0.1527073420971456) params={'rf__n_estimators': 400, 'rf__min_samples_split': 8, 'rf__min_samples_leaf': 8, 'rf__max_samples': 0.5, 'rf__max_features': 0.7, 'rf__max_depth': 6, 'rf__bootstrap': True}`  
`[sweep] train_model still running... 1920.0s elapsed`  
`target_Emerging_h6: hist_gradient_boosting CV MAE 1.106475 (std 0.15169666721069794) params={'min_samples_leaf': 20, 'max_leaf_nodes': 127, 'max_depth': 5, 'max_bins': 127, 'learning_rate': 0.03, 'l2_regularization': 0.01}`  
`[sweep] train_model still running... 1980.0s elapsed`  
`target_Emerging_h6: xgboost CV MAE 1.108024 (std 0.13242407759769395) params={'subsample': 0.8, 'reg_lambda': 1.0, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.03, 'colsample_bytree': 0.8}`  
`target_Emerging_h6: mlp_regressor CV MAE 1.438464 (std 0.19395657300043698) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 1e-05, 'mlp__activation': 'tanh'}`  
`target_Emerging_h6: selected xgboost with CV MAE 1.108024`  
`Saved model -> models\model_target_Emerging_h6.pkl`  
`Train MAE: 0.209950`  
`Val MAE: 1.082416`  
`[sweep] train_model still running... 2040.0s elapsed`  
`target_RealEstate_h6: random_forest CV MAE 1.184457 (std 0.29594522686565033) params={'rf__n_estimators': 200, 'rf__min_samples_split': 4, 'rf__min_samples_leaf': 8, 'rf__max_samples': 0.6, 'rf__max_features': 0.7, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_RealEstate_h6: hist_gradient_boosting CV MAE 1.183061 (std 0.2962071165872977) params={'min_samples_leaf': 10, 'max_leaf_nodes': 15, 'max_depth': 7, 'max_bins': 127, 'learning_rate': 0.05, 'l2_regularization': 0.05}`  
`[sweep] train_model still running... 2100.0s elapsed`  
`[sweep] train_model still running... 2160.0s elapsed`  
`target_RealEstate_h6: xgboost CV MAE 1.170399 (std 0.29431981858152) params={'subsample': 0.6, 'reg_lambda': 0.1, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_RealEstate_h6: mlp_regressor CV MAE 1.443400 (std 0.23514789048595758) params={'mlp__learning_rate_init': 0.0005, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.001, 'mlp__activation': 'relu'}`  
`target_RealEstate_h6: selected random_forest with CV MAE 1.184457`  
`Saved model -> models\model_target_RealEstate_h6.pkl`  
`Train MAE: 0.738416`  
`Val MAE: 0.979138`  
`[sweep] train_model still running... 2220.0s elapsed`  
`target_Energy_h6: random_forest CV MAE 1.853952 (std 0.7311358946696127) params={'rf__n_estimators': 600, 'rf__min_samples_split': 4, 'rf__min_samples_leaf': 2, 'rf__max_samples': 0.6, 'rf__max_features': 0.7, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_Energy_h6: hist_gradient_boosting CV MAE 1.862493 (std 0.7252810481482008) params={'min_samples_leaf': 10, 'max_leaf_nodes': 127, 'max_depth': 9, 'max_bins': 127, 'learning_rate': 0.03, 'l2_regularization': 0.001}`  
`[sweep] train_model still running... 2280.0s elapsed`  
`[sweep] train_model still running... 2340.0s elapsed`  
`target_Energy_h6: xgboost CV MAE 1.810907 (std 0.6800851811560764) params={'subsample': 0.6, 'reg_lambda': 0.01, 'min_child_weight': 5, 'max_depth': 9, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_Energy_h6: mlp_regressor CV MAE 2.397355 (std 0.6960494451628253) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (64, 32), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.0001, 'mlp__activation': 'tanh'}`  
`target_Energy_h6: selected hist_gradient_boosting with CV MAE 1.862493`  
`Saved model -> models\model_target_Energy_h6.pkl`  
`Train MAE: 0.516623`  
`Val MAE: 1.843242`  
`[sweep] train_model still running... 2400.0s elapsed`  
`target_Tech_h6: random_forest CV MAE 0.537488 (std 0.0952702803554763) params={'rf__n_estimators': 600, 'rf__min_samples_split': 4, 'rf__min_samples_leaf': 2, 'rf__max_samples': 0.6, 'rf__max_features': 0.7, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_Tech_h6: hist_gradient_boosting CV MAE 0.581674 (std 0.11524343202674844) params={'min_samples_leaf': 10, 'max_leaf_nodes': 127, 'max_depth': 7, 'max_bins': 63, 'learning_rate': 0.03, 'l2_regularization': 0.0}`  
`[sweep] train_model still running... 2460.0s elapsed`  
`[sweep] train_model still running... 2520.0s elapsed`  
`target_Tech_h6: xgboost CV MAE 0.524967 (std 0.07626194828031979) params={'subsample': 0.6, 'reg_lambda': 0.1, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_Tech_h6: mlp_regressor CV MAE 1.191695 (std 0.29562239118172023) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.001, 'mlp__activation': 'tanh'}`  
`target_Tech_h6: selected xgboost with CV MAE 0.524967`  
`Saved model -> models\model_target_Tech_h6.pkl`  
`Train MAE: 0.043972`  
`Val MAE: 0.561253`  
`[sweep] train_model still running... 2580.0s elapsed`  
`target_Financials_h6: random_forest CV MAE 1.055257 (std 0.1658050199325696) params={'rf__n_estimators': 600, 'rf__min_samples_split': 6, 'rf__min_samples_leaf': 4, 'rf__max_samples': 0.8, 'rf__max_features': 0.5, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_Financials_h6: hist_gradient_boosting CV MAE 1.050272 (std 0.16303150740278619) params={'min_samples_leaf': 20, 'max_leaf_nodes': 15, 'max_depth': 7, 'max_bins': 255, 'learning_rate': 0.05, 'l2_regularization': 0.001}`  
`[sweep] train_model still running... 2640.0s elapsed`  
`target_Financials_h6: xgboost CV MAE 1.037054 (std 0.15945217803294479) params={'subsample': 0.8, 'reg_lambda': 0.01, 'min_child_weight': 1, 'max_depth': 7, 'learning_rate': 0.03, 'colsample_bytree': 0.6}`  
`target_Financials_h6: mlp_regressor CV MAE 1.394898 (std 0.2155465335083432) params={'mlp__learning_rate_init': 0.0005, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.0001, 'mlp__activation': 'relu'}`  
`target_Financials_h6: selected random_forest with CV MAE 1.055257`  
`Saved model -> models\model_target_Financials_h6.pkl`  
`Train MAE: 0.462615`  
`Val MAE: 0.805753`  
`[sweep] train_model still running... 2700.0s elapsed`  
`target_HealthCare_h6: random_forest CV MAE 0.951869 (std 0.1236800184105084) params={'rf__n_estimators': 600, 'rf__min_samples_split': 4, 'rf__min_samples_leaf': 2, 'rf__max_samples': 0.6, 'rf__max_features': 0.7, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_HealthCare_h6: hist_gradient_boosting CV MAE 0.966102 (std 0.07902627469888798) params={'min_samples_leaf': 10, 'max_leaf_nodes': 127, 'max_depth': 9, 'max_bins': 127, 'learning_rate': 0.03, 'l2_regularization': 0.001}`  
`[sweep] train_model still running... 2760.0s elapsed`  
`[sweep] train_model still running... 2820.0s elapsed`  
`target_HealthCare_h6: xgboost CV MAE 0.954982 (std 0.12479982492585953) params={'subsample': 0.6, 'reg_lambda': 0.1, 'min_child_weight': 1, 'max_depth': 5, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_HealthCare_h6: mlp_regressor CV MAE 1.183637 (std 0.13810798621603376) params={'mlp__learning_rate_init': 0.0002, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.001, 'mlp__activation': 'tanh'}`  
`target_HealthCare_h6: selected hist_gradient_boosting with CV MAE 0.966102`  
`Saved model -> models\model_target_HealthCare_h6.pkl`  
`Train MAE: 0.300381`  
`Val MAE: 0.970536`  
`[sweep] train_model still running... 2880.0s elapsed`  
`target_Utilities_h6: random_forest CV MAE 1.136036 (std 0.21877613636625493) params={'rf__n_estimators': 600, 'rf__min_samples_split': 4, 'rf__min_samples_leaf': 2, 'rf__max_samples': 0.6, 'rf__max_features': 0.7, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_Utilities_h6: hist_gradient_boosting CV MAE 1.168254 (std 0.20912322666153876) params={'min_samples_leaf': 50, 'max_leaf_nodes': 127, 'max_depth': 9, 'max_bins': 255, 'learning_rate': 0.03, 'l2_regularization': 0.01}`  
`[sweep] train_model still running... 2940.0s elapsed`  
`[sweep] train_model still running... 3000.0s elapsed`  
`target_Utilities_h6: xgboost CV MAE 1.145577 (std 0.19897383169210375) params={'subsample': 0.6, 'reg_lambda': 0.01, 'min_child_weight': 5, 'max_depth': 9, 'learning_rate': 0.05, 'colsample_bytree': 1.0}`  
`target_Utilities_h6: mlp_regressor CV MAE 1.405578 (std 0.2564153337637568) params={'mlp__learning_rate_init': 0.0005, 'mlp__hidden_layer_sizes': (128, 64), 'mlp__batch_size': 'auto', 'mlp__alpha': 0.001, 'mlp__activation': 'relu'}`  
`target_Utilities_h6: selected random_forest with CV MAE 1.136036`  
`Saved model -> models\model_target_Utilities_h6.pkl`  
`Train MAE: 0.593047`  
`Val MAE: 1.283489`  
`[sweep] train_model still running... 3060.0s elapsed`  
`target_Industrials_h6: random_forest CV MAE 0.858475 (std 0.22064907296642175) params={'rf__n_estimators': 600, 'rf__min_samples_split': 6, 'rf__min_samples_leaf': 4, 'rf__max_samples': 0.8, 'rf__max_features': 0.5, 'rf__max_depth': 12, 'rf__bootstrap': True}`  
`target_Industrials_h6: hist_gradient_boosting CV MAE 0.915115 (std 0.21484513921891266) params={'min_samples_leaf': 10, 'max_leaf_nodes': 127, 'max_depth': 7, 'max_bins': 63, 'learning_rate': 0.03, 'l2_regularization': 0.0}`