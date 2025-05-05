import React, { PureComponent } from 'react';
import { Image, Text, TextInput, TouchableOpacity, View } from 'react-native';
import Images from '../../Theme/Images';
import { style } from './style';

interface CustomTopNavProps {
    title?: string;
    subTitle?: string;
    back?: boolean;
    onPressBack?: () => void;
    isShowCard?: boolean;
    imageName?: any;
    backgroundColor?: string;
    smallHeader?: boolean;
    largeTitle?: boolean;
    showSearchBar?: boolean;
    onSearchTextChange?: (text: string) => void;
    onPressFilter?: () => void;
    onPressSort?: () => void;
}

interface CustomTopNavState {
    searchText: string;
}

export default class CustomTopNav extends PureComponent<CustomTopNavProps, CustomTopNavState> {
    constructor(props: CustomTopNavProps) {
        super(props);
        this.state = {
            searchText: '',
        };
    }

    handleSearchChange = (text: string) => {
        this.setState({ searchText: text });
        if (this.props.onSearchTextChange) {
            this.props.onSearchTextChange(text);
        }
    };

    render() {
        return (
            <View
                style={[
                    { ...style.container, height: this.props.smallHeader ? 80 : 150 },
                    this.props.isShowCard ? style.containerShadow : style.container,
                ]}
            >
                <View style={[this.props.smallHeader ? style.smallHeaderText : style.textContainer]}>
                    {this.props.back ? (
                        <TouchableOpacity style={{ marginTop: this.props.largeTitle ? 5 : 0 }} onPress={this.props.onPressBack}>
                            <Image source={Images.backArrowImage} />
                        </TouchableOpacity>
                    ) : (
                        <Text style={style.titleText}>{this.props.title}</Text>
                    )}
                    <Text
                        style={{
                            ...style.subTitleText,
                            fontSize: this.props.largeTitle ? 30 : 20,
                            marginLeft: this.props.smallHeader ? 30 : 0,
                            marginTop: this.props.smallHeader ? 0 : 10,
                        }}
                        numberOfLines={2}
                    >
                        {this.props.subTitle}
                    </Text>
                </View>

                {this.props.showSearchBar && (
                    <View style={style.searchContainer}>
                        <TextInput
                            style={style.searchInput}
                            placeholder="Search"
                            value={this.state.searchText}
                            onChangeText={this.handleSearchChange}
                        />
                        <TouchableOpacity onPress={this.props.onPressFilter}>
                            <Image source={Images.filterIcon} style={style.iconStyle} />
                        </TouchableOpacity>
                        <TouchableOpacity onPress={this.props.onPressSort}>
                            <Image source={Images.sortIcon} style={style.iconStyle} />
                        </TouchableOpacity>
                    </View>
                )}

                <View style={style.imageContainer}>
                    <Image source={this.props.imageName} style={style.imageStyle} />
                </View>
            </View>
        );
    }
}

